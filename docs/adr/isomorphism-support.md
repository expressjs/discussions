# ADR: Export the libraries specifically in ESM format

## Status

Proposed

## Submitters

- @kjugi
- @wesleytodd
- @ctcpip

## Decision Owners

- @expressjs/express-tc

## Context

The document's objective is to gather all notable comments and thoughts in one place and track potential changes in this topic. We have noticed that it is repeated frequently in many issues from the community and we need to take action.

We have acknowledged the need and discussion around it touched on multiple scenarios. Including:
- rethinking the process and exposing both options (ESM and CommonJS) for all libraries
- expose both options (ESM and CommonJS) for selected libraries
- keeping default settings as the main target is on the server

**Why do we need this decision?**
We aimed to consolidate the Technical Committee's (TC) opinion on this topic. It is important to emphasize that Express is an HTTP framework specifically designed for Node.js. Over the years, technology has evolved, and new runtimes have emerged. Additionally, some of our libraries are being utilized by the community in other environments, such as browsers.

**What problem does it solve or avoid?**
Ambiguity and uncertainty for the community, alongside clear guidance for repository maintainers and contributors.

**Are there any existing issues/discussions/pull requests related to this?**
- https://github.com/pillarjs/router/issues/128
- https://github.com/expressjs/discussions/issues/297
- https://github.com/expressjs/express/discussions/6051
- https://github.com/jshttp/cookie/issues/211

## Decision

During [working session](https://github.com/expressjs/discussions/issues/320) we have decided to not revisit, investigate or discuss this topic further. That means ESM exports won't be available for expressjs as well for pillarjs and jshttp packages.

**What will be done?**

Future issues can be closed with a link to this document.

## Rationale

CommonJS is the default node.js syntax. The JS world moved in the ESM direction as browsers consumed it well, and bundlers could make a tree-shake feature and dynamic imports. There is still a lot of baggage within the ESM itself and the way how we can use it. 

To keep it short it's a whole new chapter to discuss and consider so it could use a lot of time and resources to make it properly. Don't get this wrong - it's not impossible tho! Most of our users will use the package in their project and pass it to the bundler which will produce the right format without any issues.

- **Alternatives Considered:**
  - Alternative 1: Add ESM export to our libraries. CommonJS format is accepted by all most popular bundlers.
- **Pros and Cons**: Outline the pros and cons of the chosen solution.
- **Why is this decision the best option?** Time and energy can be shifted to other topics.

## Consequences

- **Positive Impact**: It does not require to support another set of tools and one more major (or at least big) release.
- **Negative Impact**:
  - Packages can't be used in deno projects and potentially in other future runtime engines for JavaScript that decide to not support commonjs. That can be a potential user miss
  - OSS library authors that use our packages in ESM native libs might suffer from a lack of support
- **Mitigations**: Potential decision update to support isomorphism for selected libraries (not specified yet) and exposing both types (CJS and ESM)

## References

Support for commonjs imports in ESM code is available in the node. Described in docs:
- https://nodejs.org/api/esm.html#interoperability-with-commonjs

Support for ESM modules imports in commonjs is available since node v20 behind the experimental flag and node v23 without a flag. Docs:
- https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require

## Changelog

Track changes or updates to this ADR over time. Include the date, author, and a brief description of each change.

- **[2025-01-15]**: [@kjugi] - document init
