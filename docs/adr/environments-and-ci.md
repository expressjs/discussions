# ADR 35580f0d-f429-412b-acef-83655e3cab11: Runtime/engine/host/environment support and CI

## Status

Proposed

## Submitters

- @ctcpip

## Decision Owners

- @expressjs/express-tc

## Context

Express and its libraries were specifically designed to run with Node.js (V8). While some of our libraries can run in other environments (e.g. runtimes, engines, browsers), they are not necessarily supported in all environments. Consequently, our CI systems do not include other environments as part of their testing workflows.

Several points were raised during the discussion:

- Cost of running additional CI vs the likelihood of detecting a problem
- Introducing maintenance overhead and possible coupling to other environments' development lifecycle
- No JS engine implements ECMAScript 100% correctly; thus, claiming "ES2015 support" does not guarantee correctness across all environments.
- Environment regressions or language edge cases could break functionality in unpredictable ways that are not practical for us to monitor across all environments.

## Decision

- We will **not** add non-Node.js environment testing to our CI pipelines.
- CI will continue to run only against supported Node.js versions.
- Support for other environments may exist, but we do not guarantee correctness or compatibility across all environments.
  - Some libraries, particularly language-only libraries which do not require non-language APIs, strive to support as many environments as possible.
    - Nonetheless, support is not guaranteed across every possible environment, and is provided on a best-effort basis.
  - Libraries will not explicitly list all supported environments; they may, however, state general compatibility information, e.g. ECMAScript version.
- If issues are reported for other environments, maintainers may investigate at their discretion, but no automated validation or regression testing infrastructure will be built for them.

## Rationale

- **Alternatives Considered:**
  - **Add support for additional environments in CI**: Rejected due to complexity and minimal return on value.

- **Pros and Cons**:

  **Pros**:
  - Keeps CI lightweight and maintainable
  - Avoids implicit endorsement of non-Node.js environments
  - Maintains focus on Express's design goals and core user base

  **Cons**:
  - Some users may misinterpret lack of other environment testing as non-support
  - Manual verification may be required when bugs are reported in other environments

- **Why is this decision the best option?**
  - It balances clarity, project focus, and contributor/maintainer effort. It avoids premature optimization or expanding scope into formal environment support.

## Consequences

- **Positive Impact**:
  - Reduced CI runtime and maintenance burden
  - Clearer expectations about what we support and test

- **Negative Impact**:
  - Users of alternate environments may find compatibility issues undetected until runtime

- **Mitigations**:
  - Update README or documentation to clarify compatibility expectations and language feature dependencies
  - Encourage users of alternate environments to report issues with enough detail to investigate

## Implementation

- Close PRs proposing CI additions for alternate environments unless there is strongly compelling, project-aligned justification
- Optionally, update documentation for relevant libraries to clarify assumptions

## References

- [https://test262.fyi](https://test262.fyi)
- [path-to-regexp issue on old browser support](https://github.com/pillarjs/path-to-regexp/issues/330)
- [ADR: CommonJS and ESM](https://github.com/expressjs/discussions/pull/323)
