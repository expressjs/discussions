# ADR [Number]: LTS Strategy and Support Schedule

## Status

Proposed

## Submitters

- @wesleytodd

## Decision Owners

- @expressjs/express-tc

## Context

Conversation about our LTS strategy, support dates for our major release lines, and how we should message this content
has been spread out across multiple issues, repos, slack channels, and meetings. We have landed a few starter documents,
and generally have agreed on our goals/strategy but we have not officially documented and centralized that. This ADR is
both gathering all that context to one place as well as give concrete decisions and next steps.

### Scope:

This ADR applies to, and should cover, all repositories within the Express Projects three orgs (`jshttp`, `pillarjs`,
`expressjs`). While everything stems from support for the primary `express` package, due to the coupling of many of our
dependency and middleware packages, we must also address those packages as well. 

### ADR Goals:

- Improve documentation on our support goals
- Document criteria and reasoning for when to release new majors
- Define life cycle for releases, terminology we will use, 
- Improve documentation on how our LTS relates to Node.js version support
- Documentation on how our LTS relates to our direct dependencies and other Express project repos/packages
- Set clear next steps for documenting and publicising the LTS strategy 
- Select dates for known release line support and EOL
- Create starter language and graphics for our future blog posts, website additions, and other necessary docs 

### ADR Non-Goals:

- Create an end-user documentation resource (either for immediate use or future use)
- Commit to future release dates

**Existing issues/discussions/pull requests related to this:**

- Initial LTS strategy doc: https://github.com/expressjs/discussions/blob/master/docs/LTS-strategy.md
- Additional LTS doc proposal: https://github.com/expressjs/admin/pull/3
- https://github.com/ctcpip/express-release
- Some other related discussions:
  - https://github.com/expressjs/discussions/issues/196
  - https://github.com/expressjs/discussions/issues/172
  - https://github.com/expressjs/discussions/issues/67
- TC Slack Discussion: https://openjs-foundation.slack.com/archives/C06KMCETHG9/p1724863852681889

### Package/Repository Types

We maintain four primary types of packages/repos:

1. `express`: The minimal http framework
2. `express` dependencies: Libraries which are used directly or transitively within `express` and also by others outside
   of the project
3. Middleware: Libraries for use *with* `express` but not installed with it directly
4. Others: Tools, documentation, websites, resources, etc.

While much of this proposal is targeted specifically at `express`, this terminology will be used to differentiate where
we are not only talking about `express`.

## Decision

The Express project will have a well defined Long Term Support (LTS) strategy and schedule based on the following
requirements:

- `express` major versions will go through three supported phases: Current, Active, & Maintenance
  - `CURRENT`: A new major version is designated as `CURRENT` upon release. It is available but not the `latest` version
    on npm for a minimum of 3 months.
  - `ACTIVE`: After the minimum 3 month period and the TC agrees it is stable and secure, the `ACTIVE` version is
    tagged `latest` on npm for a minimum of 12 months.
  - `MAINTENANCE`: When a new major version becomes `ACTIVE`, the previous major version enters `MAINTENANCE` for 12 months.
- After the `MAINTENANCE` time has ended the major version is considered `EOL` and is unsupported.
- During the 12 month `ACTIVE` period, a new major version may be released but *not* until at least 6 months have passed
  since the `CURRENT` version became `ACTIVE`.
- Users are required to follow the head/latest of each major release line for support with all packages (`express`,
  dependencies, middleware, & tools/other)
- If we have no necessary breaking changes, we will not release a new major version. Applies to `express`,
  dependencies, middleware, & tools/other.
- Dropping old Node.js versions *alone* is not enough of a breaking change to release a new major version. Applies to
  `express`, dependencies, middleware, & tools/other.
- `express` version 4.x is considered a special case. It will receive a longer `MAINTENENCE` phase of 18 months or until
  the TC agrees we are save cutting support.
- `express` dependency packages will follow the same timeline and support dates for the `express` version which they are
  included with.
- Middleware packages should follow the timeline and support dates for the `express` versions they are compatible, but
  should make reasonable and informed decisions which can deviate if necessary.
- All other packages are suggested to define an appropriate strategy for support.

### Node.js Version Support

- Each major version of all packages (`express`, deps, middleware, etc) will support a minimum of the oldest Node.js LTS
at the time of release and maintain that support through its entire life cycle. This includes EOL Node.js versions.
- `express` may choose to continue support for EOL Node.js versions in all three phases if deemed valuable, but will
strive to cut support before it becomes a drain on progress.
- All `express` dependency packages must support, at a minimum, the Node.js versions that are supported by the `express`
major line they are included in while that `express` line is not `EOL`.
- Middleware packages must at a minimum follow the same Node.js versions as the `CURRENT` and `ACTIVE` release lines of
`express`. They should attempt to also support `MAINTENANCE` lines as well, but the discretion of the captains is
important.
- All other packages are suggested to follow a reasonable Node.js support policy for their needs.

### Life Cycle Phases & Meaning

**CURRENT**

- New majors will go through a short period hardening to ensure stability, security, and ecosystem libraries/resources
  compatibility.
- We will strive to ensure no breaking changes are included, but reserve the right to make security or high priority
  fixes of breaking nature within this period.
- `CURRENT` lines will receive all types of active work including: bug fixes, security patches, new features, and
  deprecation notices.
- Users are recommended to use  `CURRENT` lines and to upgrade as quickly as their risk profile allows

**ACTIVE**

- `ACTIVE` lines will receive all types of active work including: bug fixes, security patches, new features, and
  deprecation notices.
- For users, `ACTIVE` lines are considered the most stable and well supported version at any given time.

**MAINTENANCE**

- `MAINTENANCE` lines will only receive security patches or high priority bug fixes.
- Users are highly encouraged to upgrade to a `CURRENT` or `ACTIVE` release.

### Guidelines for other packages/repos

**`express` dependencies**

### Schedule

For the existing release lines, we will set the following phase dates:


| Major | CURRENT | ACTIVE | MAINTENANCE | EOL |
| ----- | ------- | ------ | ----------- | --- |
| 4.x   |         |        | 2025-04-01 | 2026-10-01 or later |
| 5.x   | 2024-09-11 | 2025-03-31 | 2026-04-01 | 2027-04-01 |
| 6.x   | TBD after 2025-10-01 | | | |

### Documentation

The project will maintain two types of documentation for this:

**Project documentation for the maintainers:**

This includes this doc, but also detailed things about the process that are not necessary for anyone but us. This
documentation will live in one of three places:

1. The `discussions` repo. If the content applies to more than one repo/package in any of the three orgs, is general in
   nature, or otherwise has no logical other place to go it goes here.
2. The individual repo/package to which it applies. If an individual package has more detailed or different needs the
   docs will live along with the repo as decided by the captains.

**User documentation:**

This will be produced for the website. While the source content of it may live in other places (for example a repo to
produce an image for the schedule), the users will be directed to the website to view the current updated version.

**Next steps:**

- Update the existing LTS doc with more details from this decision
- Update website pages with more details from this decision
- Close out discussions and other PRs related to this

## Changelog

Track changes or updates to this ADR over time. Include the date, author, and a brief description of each change.

- **[2025-02-28]**: [@wesleytodd] - Initial draft
