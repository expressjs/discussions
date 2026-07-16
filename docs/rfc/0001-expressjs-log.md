# Adopt a shared logging facade (`@expressjs/log`) across the Express ecosystem

## Summary

Introduce `@expressjs/log`, a tiny zero-dependency logging facade owned by the Express project, and migrate `express`, `router`, `send`, `finalhandler`, and `body-parser` off the `debug` and `depd` packages to it in their next majors (deprecations flow through a `deprecate()` helper that keeps Node's native `DeprecationWarning` semantics). The facade defines no new logging API. Its contract is the intersection of [pino](https://getpino.io) and the proposed Node.js core logger ([nodejs/node#60468](https://github.com/nodejs/node/pull/60468)): six RFC 5424-ordered level methods, child loggers, and enabled-level checks. By default it is silent below `error` (near-zero per-call cost) and writes `error`/`fatal` records to stderr, preserving express's long-standing behavior of printing unhandled request errors; applications opt in to more by registering a single consumer (a pino instance, `console`, or any object implementing the contract), and diagnostic tooling can subscribe through a `diagnostics_channel` without any configuration.

## Motivation

- **Dependency footprint and supply-chain risk.** Every production install of `express` today pulls `debug` (plus `ms`) and `depd` through five packages ([expressjs/discussions#459](https://github.com/expressjs/discussions/issues/459)). Both are externally maintained and their maintainer has been inactive for some time. After this migration, `debug` and `depd` leave the production tree entirely: not conditionally loaded, simply not installed.
- **Unstructured, framework-private diagnostics.** `debug` emits interpolated text filtered by the `DEBUG` env var. Applications cannot route framework diagnostics into their own logging pipeline (pino, winston, an APM), cannot filter by severity, and cannot consume the data programmatically. This has come up repeatedly ([expressjs/discussions#341](https://github.com/expressjs/discussions/issues/341), [expressjs/express#6280](https://github.com/expressjs/express/issues/6280), [expressjs/express#6353](https://github.com/expressjs/express/issues/6353), [pillarjs/router#151](https://github.com/pillarjs/router/pull/151)).
- **One integration point for the whole ecosystem.** Today each package makes its own logging decision. A shared facade means an application configures logging once and every participating package flows through it, and it means underlying implementations (a pino major, `node:logger` API churn before stabilization) can change inside the facade without a breaking change in any consuming package, avoiding the pass-through-API mistakes of the past.

Expected outcome: an application that configures nothing pays the cost of loading one trivial, org-owned module and an integer comparison per disabled log call. An application that runs `setConsumer(pino())` gets structured, leveled framework diagnostics in its own pipeline. An APM gets the same via `diagnostics_channel` with zero app changes.

## Detailed Explanation

### Module surface

```js
const log = require('@expressjs/log')
// `log` is the root Logger. Named exports:
const { setConsumer, deprecate, StderrConsumer, LEVELS, CHANNELS } = require('@expressjs/log')
```

```ts
type Level = 'trace' | 'debug' | 'info' | 'warn' | 'error' | 'fatal'
type Fields = Record<string, unknown>

interface LogFn {
  (msg: string, ...args: unknown[]): void
  (fields: Fields, msg?: string, ...args: unknown[]): void
  (err: Error, msg?: string, ...args: unknown[]): void
  readonly enabled: boolean
}

interface Logger {
  readonly trace: LogFn
  readonly debug: LogFn
  readonly info: LogFn
  readonly warn: LogFn
  readonly error: LogFn
  readonly fatal: LogFn
  child(bindings: Fields): Logger
  readonly bindings: Readonly<Fields>
}
```

The root logger is frozen. There is no `new Logger()` in the public surface: framework packages only ever derive children from the root, so the facade's state (active consumer, effective level) is process-global and unambiguous.

### Levels

Numeric values are pino-compatible and RFC 5424-ordered, matching `node:logger`: trace=10, debug=20, info=30, warn=40, error=50, fatal=60. Exported as `LEVELS` (frozen name→number map). There is no facility for custom levels: that is implementation territory, and the facade deliberately defines no API richer than the pino/`node:logger` intersection.

### Call semantics

`log.<level>([fields | err], [msg], ...args)`

1. **First-argument dispatch:** a string is the message; a plain object is `fields`; an `Error` instance is shorthand for `({ err }, msg ?? err.message)`.
2. **Formatting:** `msg` is formatted with `util.format(msg, ...args)` semantics (`%s %d %j %o`, surplus args appended). Formatting happens **only if the level is enabled**; the facade owns lazy evaluation.
3. **Reserved keys:** `level`, `time`, and `msg` on the record are facade-owned. A `msg` key inside `fields` is used only when no message argument is given (this is the `node:logger` calling convention; accepting it keeps hand-migrated code portable). User fields may not override `level`/`time`; colliding keys are dropped.
4. **`err` convention:** an `Error` under the `err` key is serialized to `{ type, message, stack, code?, ...ownEnumerable }` before reaching consumers that don't handle Errors natively. Consumers that do (pino ships error serializers; the `node:logger` proposal includes them) receive the original `Error`; the built-in stderr consumer serializes.
5. **Never throws:** serialization or consumer failures are swallowed; built-in consumers emit a best-effort record with an `"[unserializable]"` placeholder on failure. A logging call must never take down a request.

### `child(bindings)`

- Shallow-merges bindings; child keys win over parent keys; call-site fields win over both.
- Children are immutable and cheap to create (no consumer interaction or serialization at creation), so per-request children like `log.child({ reqId })` are viable for applications, though framework packages only create module-level children.
- Children created before or after `setConsumer()` behave identically: level state lives in shared facade state, not copied into the child.

**Framework convention** (replaces `debug` namespaces): every package creates module-level children with `pkg` and `mod` bindings.

```js
// lib/view.js in express
const log = require('@expressjs/log').child({ pkg: 'express', mod: 'view' })

log.debug('lookup %j', name)                 // was: debug('lookup "%s"', name)
log.debug({ path: view.path }, 'render')     // data goes in fields, not in msg
```

Namespace mapping: `express:view` → `{ pkg: 'express', mod: 'view' }`, `router:layer` → `{ pkg: 'router', mod: 'layer' }`, `send` → `{ pkg: 'send' }`.

### `.enabled`

`log.debug.enabled` is a boolean reflecting the effective level of the active consumer, and the guard for expensive field construction:

```js
if (log.debug.enabled) log.debug({ stack: buildStackSnapshot(app) }, 'stack rebuilt')
```

Cost contract: a **disabled** level call (no consumer at that level, no channel subscriber) is one integer comparison plus one boolean property read and an early return; `.enabled` is one property read. This is the `debug`-parity requirement, and it is why level filtering happens in the facade rather than in the consumer. Matches `node:logger`'s `logger.info.enabled`; maps to `isLevelEnabled()` on pino.

### The `LogRecord`

What consumers receive. Flat and pino-line-compatible, so `JSON.stringify(record)` is a valid NDJSON line ingestible by the existing pino tooling ecosystem:

```ts
interface LogRecord {
  level: number            // 10..60
  time: number             // epoch ms
  msg?: string             // already formatted
  pkg?: string             // from bindings
  [key: string]: unknown   // remaining bindings + call-site fields, flattened
}
```

Precedence (last wins): parent bindings → child bindings → call-site fields → facade-owned `level`/`time`/`msg`.

### Consumer contract

```ts
interface Consumer {
  level?: Level                 // minimum level, default 'info'
  handle(record: LogRecord): void
}

setConsumer(target: Consumer | LoggerLike | Console | null): previous
```

This RFC proposes `setConsumer` as process-global, with no per-app injection in v1. Per-app injection (`express({ logger })`) *is* implementable for express core itself (every express-core call site sees the app, so an app-scoped child logger is straightforward), but it cannot reach `router`, `send`, or `finalhandler`, which never see the app object; extending it would require plumbing logger options through every constructor in the stack. The result would be a partial integration that misleads: a user who passes `express({ logger })` expects framework logs and would get boot/settings/view records while the diagnostics they actually came for (route matching, body parsing, static file resolution) keep flowing elsewhere. One well-documented global seems preferable to that foot-gun. Two further reasons favor global-only: framework diagnostics are process-level concerns (there is one `express` module instance per process, however many apps it serves), and the single-global-binding model is the proven shape of every logging facade in Prior Art (SLF4J, Rust `log`). An application that needs per-app separation can do it in its consumer, since every record carries the bindings to discriminate on. `express({ logger })` remains available as a follow-up (sugar scoping a child over the global) if demand materializes once the ecosystem is on the facade.

`setConsumer` is last-write-wins and returns the previously installed target (so tests can restore). **Only applications may call it; framework packages never do.** Accepted forms:

1. **`Consumer`** (anything with a `handle` function): used directly. The escape hatch for custom sinks.
2. **Logger-like** (has the six level methods: pino, a `node:logger` instance, winston behind a level-method shim): the facade adapts by calling `target[levelName](fields, msg)`, passing raw `Error`s through under `err`. Effective level is read from `target.level` when present, else `target.isLevelEnabled?.(name)` per level, else `'info'`.
3. **`console`**: trace/debug → `console.debug`, info → `console.info`, warn → `console.warn`, error/fatal → `console.error`, rendered as `express [mod] msg {fields}`.
4. **`null`**: factory reset, as if `setConsumer` had never been called. The facade re-resolves its default: the `NODE_DEBUG` shim if that env var is set, otherwise the built-in errors-to-stderr consumer. Note this is different from silencing: `setConsumer(null)` brings default error output *back*, while `setConsumer({ handle() {} })` (a noop consumer) turns everything off.

**Conflict semantics.** `setConsumer` is a process singleton and conflicts must be loud, not silent. Replacing an *explicitly installed* consumer (not the default) emits a `process.emitWarning` (`ExpressLogWarning`) carrying the installation stacks of both parties, so an application whose logging sink gets hijacked by a dependency sees exactly who did it. First-write-wins (Rust `log`'s model) does not work here: dependency module init runs before the application's entry-point code, so a misbehaving dependency would win the race, the opposite of the intent. The rule for dependencies is documented the same way undici documents `setGlobalDispatcher`: **libraries never call `setConsumer`.** A dependency that wants to *observe* framework logs has a first-class, conflict-free path: subscribe to the diagnostics channel (N subscribers, no ownership). The returned previous consumer exists for tests and for deliberate cooperative wrapping (a tee that delegates to the previous sink), not as a mechanism for libraries.

Deliberately absent: multiple simultaneous consumers, per-`pkg` consumer routing, transports. One consumer; fan-out is the consumer's (or a diagnostics-channel subscriber's) problem. This is the "don't write a logger" line.

### Built-in consumers

- **default** (no configuration): levels below `error` are disabled (noop cost: one integer comparison per call); `error` and `fatal` records are written to stderr via `console.error`. This is not an arbitrary choice: it preserves express's current behavior, where unhandled request errors print by default (see *Unhandled errors* below) while all other diagnostics are silent. A fully-noop default would silently swallow unhandled errors, a regression.
- **`StderrConsumer({ level = 'debug', stream = stderr })`**: one NDJSON line per record, synchronous write, no colors, no worker threads. Exists to serve the `NODE_DEBUG` shim (see *Environment activation* below) and tests, not to compete with pino transports.

### Unhandled errors (finalhandler)

Express's only default-visible output today is `logerror`, the `onerror` hook it passes to `finalhandler`, which does `console.error(err)` for errors that reach the final handler (suppressed when `env === 'test'`). This is framework diagnostics at `error` level, and the migration moves its *ownership* to where the error is actually handled:

- **`finalhandler`'s default `onerror` becomes `log.error(err, 'unhandled error')`.** Today, standalone finalhandler users (connect, plain `http` servers) get silently swallowed errors unless they wire `onerror` themselves; with the facade default, the package that terminates the request reports the failure, and the facade's default consumer puts it on stderr.
- **Express drops `logerror` and stops passing `onerror` entirely**, inheriting finalhandler's default. This must land in the same express major that adopts the migrated finalhandler; keeping both would report every unhandled error twice. An explicit `onerror` passed by a host continues to *replace* the default, preserving finalhandler's existing option semantics.

Consequences:

- Default behavior is unchanged for applications: unhandled errors still print to stderr (via the default consumer above).
- With a consumer registered, unhandled errors become structured records in the application's own pipeline, one of the most-requested integrations (today they bypass the app's logger entirely).
- APMs see unhandled errors on the diagnostics channel with zero configuration.
- The hardcoded `env !== 'test'` suppression can be retired: test suites install a capture or noop consumer instead of relying on a magic env value. Whether to keep honoring `NODE_ENV=test` for one major as a compatibility bridge is left as an unresolved question.

### Deprecations (`depd` replacement)

Deprecation warnings also flow through this system, removing `depd` alongside `debug`. The facade exports one helper:

```js
const { deprecate } = require('@expressjs/log')

deprecate('res.redirect(): provide a url argument')
```

Semantics of `deprecate(msg)`:

1. **Once per message per process**, so a hot path never floods stderr.
2. **Native half:** emits `process.emitWarning(msg, 'DeprecationWarning')`. This is what preserves everything the platform already gives users: visible on stderr by default, `--no-deprecation`, `--trace-deprecation`, `--throw-deprecation`, `process.on('warning')`, and CI/test tooling that recognizes `DeprecationWarning`. The facade does not reimplement any of it. `process.emitWarning` exists in every supported Node line (it shipped in Node 6) and in Bun/Deno's compat layers; for non-Node runtimes where `process` is absent, the helper degrades to `console.warn` (stderr, never stdout) instead of throwing.
3. **Record half:** emits a `warn`-level record tagged `{ deprecation: true }` through the facade: registered consumers and the diagnostics channel receive deprecations as structured data in the application's own pipeline. The **built-in** consumers (default and `StderrConsumer`) skip deprecation-tagged records, because the native half already prints to stderr; without this rule every deprecation would print twice by default.

`deprecate` is the one deliberate extension beyond the pino/`node:logger` intersection contract. It is justified because it is a utility *on top of* the Logger contract, not part of it: no backend needs to know it exists, and it composes from primitives (`emitWarning` + `log.warn`) that every backend already supports.


### Environment activation

Checked once at first import, lowest priority (any `setConsumer()` call overrides):

- `NODE_DEBUG=<sections>` → `StderrConsumer` at `debug`, filtered to matching `pkg`/`mod` bindings. Sections follow Node's own convention: a comma-separated list, where each section is a `pkg` (`express`, `router`, `send`) or a `pkg:mod` pair (`express:view`, `router:layer`). So `NODE_DEBUG=express,router,send` covers the typical "show me everything the framework does" case, and `NODE_DEBUG=router:layer` narrows to one module. This is the migration shim for today's `DEBUG=express:*,router:*,send` users; documented as compatibility-only and not guaranteed to survive to v2.

There is deliberately no facade-specific env var: environment activation exists only as the `debug`-migration bridge, programmatic needs go through `setConsumer()`, and tooling needs go through the diagnostics channel, which is fully independent of both (below). Silencing the default error output is done with a noop consumer: `setConsumer({ handle() {} })`.

The shim reads `NODE_DEBUG` rather than continuing to honor `DEBUG`, deliberately. `DEBUG` and its pattern grammar (wildcards, `-` exclusions) are the public API of the package this RFC removes: honoring the variable means either reimplementing `debug`'s matcher forever or surprising users with partial support, and it would keep the retired contract alive indefinitely. It is also a generically-named variable that CI environments set for unrelated purposes, and during the multi-year transition (while unmigrated middlewares still pull `debug`) two systems answering the same variable with different output formats would interleave confusingly on stderr. `NODE_DEBUG=express` collides with nothing and follows the platform's own convention.

### Diagnostics channel

Records are published on **one `diagnostics_channel` per level**, named `express.log.<level>`:

```
express.log.trace  express.log.debug  express.log.info
express.log.warn   express.log.error  express.log.fatal
```

Per-level channels let subscribers pay only for what they want: a tool that only cares about errors subscribes to `express.log.error` and the facade never constructs `debug` records on its behalf. A subscriber that wants everything subscribes to all six (a loop over `LEVELS`). This mirrors `node:logger`'s architecture exactly (its `log:<level>` channels), which also makes the future backend substitution a one-to-one channel mapping.

The names are exported as the `CHANNELS` constant (frozen level→name map) so producer and subscribers share one source of truth instead of hardcoded magic strings. That said, `diagnostics_channel` registers channels globally by name, so tooling can also subscribe with the raw string and no `require`; this makes the channel names public API, and their exact spelling is tracked in *Unresolved Questions and Bikeshedding* (naming). Publishing is guarded per level by `channel.hasSubscribers`, so the unsubscribed cost is one boolean check. This gives APMs and tooling zero-config access to framework diagnostics, the [#341](https://github.com/expressjs/discussions/issues/341) requirement.

The ecosystem is converging on `diagnostics_channel` as the logging interop layer: `node:logger` is built on it (`log:<level>` channels), pino publishes [`tracing:pino_asJson:*` tracing-channel events](https://github.com/pinojs/pino/blob/main/docs/diagnostics.md) around its serialization, and our own org is already moving this way: [pillarjs/router#196](https://github.com/pillarjs/router/pull/196) adds an `express.router.request` TracingChannel emitting lifecycle events for every middleware and handler execution. That PR and this RFC are complementary layers of the same strategy: tracing channels carry *execution* events (spans for APMs), while the `express.log.*` channels carry *diagnostic records* (what the framework has to say). They are also complementary rather than redundant with pino's channels: pino's events exist only when a pino instance is installed and carry the already-serialized log line for the *whole application*, while `express.log.*` carries the structured framework-only `LogRecord` and works with no consumer configured at all. If the backend ever becomes `node:logger`, two channel surfaces will exist, and that is by design: `express.log.<level>` is the facade's permanent contract and **always publishes regardless of backend** (disabling it on a backend switch would silently break existing subscribers, which the frozen-names rule forbids), while `node:logger`'s own `log:<level>` channels additionally carry the records as an inherent consequence of that backend, mixed with every other `node:logger` record in the application. Subscribers pick the surface that matches their intent: `express.log.*` for framework-only diagnostics, `log:*` for application-wide core-logger output. Subscribing to both means seeing framework records twice; documentation says to choose one.

**The channels are independent of the consumer.** The record-construction condition is per level: `levelEnabled || channels[level].hasSubscribers`. With a subscriber on a level's channel, records at that level are built and published regardless of the consumer's effective level. An APM loaded via `--require` sees framework diagnostics at exactly the verbosity it subscribed to, with no consumer registered, no env vars, and nothing printed to stderr; levels nobody subscribed to are never constructed. With no subscriber and no enabled level, a call remains one integer comparison plus one boolean property read.

### Backend mapping

| Facade | pino | `node:logger` (#60468) | `console` adapter |
|---|---|---|---|
| `log.info(f, msg, ...a)` | `pino.info(f, msg, ...a)` | `logger.info({ ...f, msg: fmt })` | `console.info(rendered)` |
| `log.child(b)` | `pino.child(b)` | `logger.child(b)` | facade-side merge |
| `log.info.enabled` | `isLevelEnabled('info')` | `logger.info.enabled` | facade-side level |
| `err` field | pino `err` serializer | built-in error serializer | `stack` printed |
| record wire shape | native NDJSON line | `JSONConsumer` line | n/a |

A change in the right columns (pino major, `node:logger` API churn before stabilization) is absorbed inside `@expressjs/log` patch/minor releases without any change to the two left columns.

### Loading and runtime cost

- The facade itself always loads with `express`: a single zero-dependency file (contract, global state, default consumer), comparable to or smaller than loading `debug` today.
- **No implementation is ever loaded unless activated.** There is no `require('pino')` anywhere; pino enters only if the application installs it and passes it to `setConsumer()`. `node:logger` (if the proposal lands and stabilizes) would be a built-in, touched only when available and a consumer is active. `StderrConsumer` is only instantiated under `NODE_DEBUG`.
- With the default consumer and no channel subscriber, a `log.debug(...)` call performs one integer comparison plus one boolean check and returns: no record construction, no formatting. Only `error`/`fatal` records are constructed by default.

### Worked examples

Application wiring pino:

```js
const pino = require('pino')()
require('@expressjs/log').setConsumer(pino)
const app = express()   // framework logs now flow into the app's pino pipeline
```

Test capture (framework test suites):

```js
const records = []
const prev = setConsumer({ level: 'trace', handle: r => records.push(r) })
// ... exercise code ...
setConsumer(prev)
assert(records.some(r => r.mod === 'view' && r.msg.startsWith('lookup')))
```

Tooling subscribing without touching app configuration:

```js
const { CHANNELS } = require('@expressjs/log')
const dc = require('node:diagnostics_channel')

dc.subscribe(CHANNELS.error, reportError)                    // errors only
for (const name of Object.values(CHANNELS)) {                // or everything
  dc.subscribe(name, onRecord)
}
```

## Rationale and Alternatives

1. **Replace `debug` with `util.debuglog` directly** (original shape of [#459](https://github.com/expressjs/discussions/issues/459)). Removes the dependency with minimal code change, but keeps diagnostics as unstructured text, offers no severity levels, no application integration point, and changes the user-facing contract (`DEBUG=` → `NODE_DEBUG=`, no colors, `SECTION PID:` prefix); that is a breaking change users would likely absorb *twice* if a structured facade arrives later. The facade is one break instead of two.
2. **Accept an injectable generic logger per app** (`express({ logger })` against an `info/debug/error` duck type, as suggested in #459). Solves integration for `express` itself but not for `router`/`send`/`finalhandler`/`body-parser`, which never see the app object; it also makes the duck type (an API we'd have to freeze forever) the contract, repeating the pass-through problem. The facade keeps injection (via the process-global `setConsumer`) while owning the contract.
3. **Depend on pino directly.** Best-in-class implementation, but it puts a large external dependency in every install (the opposite of the motivation), forces its transport/worker model on all users, and couples the ecosystem's public API to pino's semver.
4. **Wait for `node:logger` and use it directly.** `node:logger` does not exist yet: it is an open, unmerged PR ([nodejs/node#60468](https://github.com/nodejs/node/pull/60468)) proposing an experimental module behind an `--experimental-logger` flag, still under TSC review; it may change shape substantially or never land at all. Even in the best case (merged, flagged, eventually stable), the ecosystem supports Node lines that will never have it, and using it directly would repeat the pass-through problem: any pre-stabilization API change would break every consuming package. The facade lets the ecosystem move now and adopts `node:logger` as a backend if and when it becomes usable, with zero changes for users.

The facade is the only option that simultaneously removes the external dependency, provides structure and levels, gives one integration point, and preserves freedom to swap implementations.

## Implementation

**New repository:** `expressjs/log`, publishing `@expressjs/log@1.x`. Single-file implementation plus types; zero runtime dependencies; standard org captain/triage model. Ships: contract, errors-to-stderr default, `StderrConsumer`, `setConsumer` + adapters, env activation, diagnostics channel.

**Consuming packages** (each in its next major, in rough dependency order):

| Package | Current | Change |
|---|---|---|
| `pillarjs/send` | `debug` (1 namespace) | swap to facade child `{ pkg: 'send' }` |
| `pillarjs/router` | `debug` (3 namespaces), `depd` | swap namespaces to children; `depd` calls → `deprecate(msg)` |
| `pillarjs/finalhandler` | `debug`; silent by default without `onerror` | swap to facade child; default `onerror` → `log.error` (see *Unhandled errors*) |
| `expressjs/body-parser` | `debug` (per-parser namespaces) | swap to children `{ pkg: 'body-parser', mod: <parser> }` |
| `expressjs/express` | `debug` in `lib/application.js`, `lib/view.js` (7 call sites); `depd` in `lib/response.js` (3 call sites); `console.error` in `logerror` (finalhandler `onerror`) | swap to children; `depd` calls → `deprecate(msg)`; drop `logerror` and inherit finalhandler's default `onerror` (see *Unhandled errors*); re-export nothing (facade is reached via its own package) |

`debug` and `depd` leave the production install only when all five have released majors; this RFC establishes the shared target so those migrations can proceed independently. The facade is general-purpose: any package that needs it can adopt it, regardless of org. `http-errors` (jshttp) uses `depd` only and could migrate to `deprecate()` whenever its maintainers choose, on their own schedule.

**Second ring (org middlewares).** Beyond the core install, these expressjs middlewares also depend on `debug` today: `express-session`, `cookie-session`, `method-override`, `morgan`, `compression`, `serve-index` (and `express-session`, `morgan`, `response-time` on `depd`, which maps to `deprecate()`). They are not blockers for the core migration (they are app-installed, not part of the `express` tree), but they should adopt the same facade child convention (`{ pkg: 'express-session' }`, …) in their next majors so that one `setConsumer()` call covers the entire official ecosystem. This is precisely the argument for the facade over per-package solutions: without a shared contract, every middleware re-decides logging on its own.

**morgan wears two hats.** As a *package*, its internal `debug` calls migrate to the facade like every other middleware. As a *product*, morgan is an HTTP access logger (application-facing output), and access log lines do **not** route through `@expressjs/log`, whose records are framework diagnostics for a different audience (framework maintainers/operators debugging express itself, vs. operators consuming request logs). Whether morgan's future is a structured rewrite or deferring to `pino-http` is a separate discussion this RFC deliberately does not open.

**User-facing breaking change** (per package, at its major): `DEBUG=express:*` stops producing output; replacement is `NODE_DEBUG=express` or a registered consumer. Migration notes go in each package's changelog plus the express major migration guide.

**Testing:** the facade repo carries contract tests (call semantics, precedence, adapter behavior, never-throws) plus adapter tests against pinned pino. Consuming packages' suites swap `DEBUG`-based assertions for the test-capture consumer shown above.

**Sequencing with `node:logger`:** no dependency on the PR landing: the facade is fully functional without it, indefinitely. If the proposal is merged and eventually stabilizes, `@expressjs/log` adopts it as the default enabled backend in a minor release; consuming packages change nothing. If it never lands, nothing in this design needs revisiting.

## Prior Art

- **SLF4J (Java).** The canonical logging facade: libraries log against a stable API, applications bind one implementation. Effectively ended per-library logger fragmentation in the Java ecosystem; this proposal is the same shape scoped to one framework's packages.
- **Rust `log` crate + `tracing`.** A facade crate with a `set_logger` global and noop default, implementations chosen by the binary. Validates the single-global-consumer + near-zero-disabled-cost design.
- **Go `log/slog`.** Structured leveled logging added to the standard library with a `Handler` (consumer) abstraction; explicitly cited as the model in [nodejs/node#49296](https://github.com/nodejs/node/issues/49296). Its default-handler + swappable-backend split matches this design.
- **pino.** Source of the record wire format, numeric levels, `child(bindings)` and `err` serialization conventions used here. Its [`tracing:pino_asJson:*` diagnostics events](https://github.com/pinojs/pino/blob/main/docs/diagnostics.md) are further evidence of `diagnostics_channel` as the ecosystem's logging interop layer.
- **`node:logger` PR ([nodejs/node#60468](https://github.com/nodejs/node/pull/60468)).** Logger/Consumer split over `diagnostics_channel`, `logger.<level>.enabled`, RFC 5424 numeric levels; this facade is designed to adopt it as a backend without API change.
- **`debug`.** Prior art for the near-zero disabled cost bar and the namespace convention this design maps onto `pkg`/`mod` bindings.

## Unresolved Questions and Bikeshedding

1. **`NODE_DEBUG` shim wildcards:** Node's own `NODE_DEBUG` supports trailing wildcards (`NODE_DEBUG=express*`). Should the shim match that (e.g. `express*` covering every `pkg`), or is the explicit comma-separated list enough?
2. **Default consumer level** when a logger-like target exposes no level: `info` (pino convention) or `warn` (quieter)?
3. **Object-with-`msg` calling convention.** The facade's canonical call shape is pino's, but `node:logger` puts the message inside the fields object, and the spec currently accepts both:

   ```js
   log.info({ userId: 123 }, 'user logged in')      // canonical (pino shape)
   log.info({ msg: 'user logged in', userId: 123 }) // node:logger shape, also accepted
   ```

   Do we document the second form as a permanent part of the contract, or as a portability affordance for code written against `node:logger`'s convention, removable in a future major? Permanent means code ports between the facade and `node:logger` unchanged, at the cost of two ways to say the same thing forever.
4. **Naming:** `@expressjs/log` vs `@expressjs/logger`; `setConsumer` vs `setLogger`; channel prefix `express.log.<level>` vs alternatives. There is org precedent to align with: [pillarjs/router#196](https://github.com/pillarjs/router/pull/196) names its tracing channel `express.router.request` (`express.<package>.<subject>`), which the `express.log.<level>` scheme follows; whatever scheme wins should be consistent across both. Note the channel names deserve the most care: observability agents subscribe with the raw strings (no import), so they are public API frozen from v1; renaming them later silently breaks subscribers.
5. **`NODE_ENV=test` error suppression:** today `logerror` is suppressed when `env === 'test'`. Keep honoring it for one major as a compatibility bridge, or drop it immediately in favor of a noop/capture test consumer?
