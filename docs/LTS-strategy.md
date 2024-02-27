# Express LTS Strategy

## Goals

* Maintain strict Node.js version support within a single Express major release.
* Provide good ergonomics and developer experience for Express maintainers
* Provide good experience for users who want to keep their Express projects that up-to-date
* Avoid stagnation and blocking innovation that is good for the end users

## Non-Goals

* Only support the active LTS releases of Node.js
* Support the widest possible set of Node.js versions. We aim to maintain a healthy balance between backwards compatibility and toolchain modernization.
* Provide timed cadence of Node.js version support dismissal

### Rationale behind dropping support for old Node.js versions

* Avoid blocking Node.js core from making progress and removing deprecated parts of the codebase
* Avoid getting outdated bug reports or security issues
* Keep options open for adopting newer patterns and tooling

### Rationale behind conservative policy of breaking changes

* Minimize disruption for end users
* Be mindful of the fact that some users are still running obsolete versions of Node in production

### Guidelines for dropping support of Node.js versions

* In order to smoothen the experience of users updating to a new semver major Express version, Express should always support at least one even numbered Node.js version above the current lowest supported one before the drop can be considered. For example, if the lowest supported version is 20, Express support for Node 20 cannot be dropped until Express supports at least Node 22. This ensures that Express updates can be done independently from the Node.js version update.
* Version support removal must come with clear benefits for one of the three - Express users, Express maintainers, or the Node.js core project. Version support is not dropped on a merely time-based basis, or otherwise arbitrarily.
* Any discontinuation of a Node.js version support is always a semver major release.

### Active branches

* Several semver major branches will be maintained at the same time while we have overlapping supported versions
* No features are backported to older semver major branches
* Critical bugfixes will be backported to older supported semver major branches
* High-impact security fixes are very likely to be backported to older semver major branches
