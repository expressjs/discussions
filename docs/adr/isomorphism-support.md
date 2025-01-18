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

During the [working session](https://github.com/expressjs/discussions/issues/320), we had an in-depth discussion about this topic. After careful consideration, we concluded that we will not make a dedicated effort to export our libraries in the ESM format. Instead, we will continue exporting the libraries as we have done historically. 

This decision is motivated by the lack of resources to maintain such an effort in the long term. It is also worth noting that Express is specifically designed to run with Node.js. While some of our libraries can be considered "isomorphic," this was unintended and can currently be classified as an "unofficial but functional feature." Consequently, our CI systems do not include browsers or other runtimes as part of their testing workflows.

At present, our libraries function seamlessly in Node.js, supporting both CommonJS and ESM. Transitioning to support additional scenarios, such as direct ESM exports, would require significant changes to our CI systems and additional maintenance overhead.

**TL;DR**: Dedicated ESM exports will not be available for Express.js, PillarJS, or JSHTTP packages. PR with such a change will not be accepted.

**What will be done?**

Future issues can be closed with a link to this document.

## Rationale

CommonJS is the default syntax in Node.js. While the JavaScript ecosystem has increasingly moved toward ESM due to its compatibility with browsers, enhanced tree-shaking capabilities, and support for dynamic imports, there are still complexities and challenges associated with ESM. 

Adopting ESM for our libraries would require a significant investment of time and resources to ensure proper implementation and long-term maintenance. While it is not impossible to achieve, it represents a considerable effort. Moreover, the majority of our users already utilize our libraries in their projects, relying on bundlers to handle the necessary transformations without issues.

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
- **[2025-01-18]**: [@kjugi] - applied code review suggestions
