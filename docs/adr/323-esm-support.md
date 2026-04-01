# ADR: ESM exports

## Status

Accepted

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
- maintaining status quo as the main target is on the server

**Why do we need this decision?**
We aimed to consolidate the Technical Committee's (TC) opinion on this topic. It is important to emphasize that Express is an HTTP framework specifically designed for Node.js. Additionally, some of our libraries are being utilized by the community in other environments.

**What problem does it solve or avoid?**
Ambiguity and uncertainty for the community, alongside clear guidance for repository maintainers and contributors.

**Are there any existing issues/discussions/pull requests related to this?**

- <https://github.com/pillarjs/router/issues/128>
- <https://github.com/expressjs/discussions/issues/297>
- <https://github.com/expressjs/express/discussions/6051>
- <https://github.com/jshttp/cookie/issues/211>

## Decision

Holistically, as a project/team, we will not make a dedicated effort to export our libraries in the ESM format.

This applies to all [expressjs](https://github.com/expressjs), [pillarjs](https://github.com/pillarjs), or [jshttp](https://github.com/jshttp) packages.

**However, repo captains, as the primary maintainers of packages, retain autonomy and discretion of whether or not they choose to support ESM exports directly.**

### What will be done?

Future issues and PRs regarding ESM exports may be closed with a link to this document.

## Rationale

During the [working session](https://github.com/expressjs/discussions/issues/320), we had an in-depth discussion about this topic.

This decision is motivated by the lack of resources to maintain such an effort in the long term. It is also worth noting that Express was specifically designed to run with Node.js.

At present, our libraries function seamlessly in Node.js, supporting both CommonJS and ESM. Transitioning to support additional scenarios, such as direct ESM exports, would require significant changes to our CI systems and additional maintenance overhead.

While the JavaScript ecosystem has increasingly moved toward ESM due to its compatibility with browsers, enhanced tree-shaking capabilities (coming from bundler tools), and support for dynamic imports, there are still complexities and challenges associated with ESM.

Adopting ESM for our libraries would require a significant investment of time and resources to ensure proper implementation and long-term maintenance. While it is not impossible to achieve, it represents a considerable effort. Moreover, the majority of our users already utilize our libraries in their projects, relying on bundlers to handle the necessary transformations without issues.

- **Alternatives Considered:**
  - Add ESM export to our libraries. CommonJS format is accepted by bundlers.
- **Why is this decision the best option?**
  - Time and energy can be shifted to other topics.

## Consequences

- **Positive Impact**: It does not require supporting another set of tools and major release.
- **Negative Impact**:
  - No guarantee the packages work in browser environments.
  - Potential community library fork (to make it ESM-friendly) might lack security updates over time
  - OSS library authors that use our packages in ESM native libs might suffer from a lack of support
- **Mitigations**: Package maintainers may decide to expose both CJS and ESM

## References

Support for commonjs imports in ESM code is available in the node. Described in docs:

- <https://nodejs.org/api/esm.html#interoperability-with-commonjs>

Support for ESM modules imports in commonjs is available in recent LTS Node.js versions. Docs:

- <https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require>

## Changelog

Track changes or updates to this ADR over time. Include the date, author, and a brief description of each change.

- **[2025-01-15] - [2026-04-01]**: [@kjugi, @ctcpip, @wesleytodd] - initial doc
