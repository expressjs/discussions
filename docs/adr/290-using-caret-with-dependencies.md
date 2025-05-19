# ADR #290: Policy on Using Caret (`^`) or tilde (`~`) with Dependencies in `package.json`

## Status

Accepted

## Submitters

- Ulises Gascón (@UlisesGascon)

## Decision Owners

- Express TC (@expressjs/express-tc)

## Context

Historically, the Express project has avoided using the caret (`^`) in the `package.json` files for its own dependencies. This proposal aims to review whether this practice should continue or if adjustments are needed.

**Why do we need this decision?**  
Clarifying the policy on using caret (`^`) helps to ensure consistency across the Express ecosystem, reduce unnecessary maintenance, and prevent unexpected dependency updates. It also addresses concerns about the balance between keeping dependencies up-to-date and avoiding regressions.

**What problem does it solve or avoid?**  
This decision aims to reduce the maintenance burden of frequently updating pinned dependencies.

**Are there any existing issues/discussions/pull requests related to this?**  

- [Discussion: Using caret (^) with our own dependencies #279](https://github.com/expressjs/discussions/issues/279)
- [expressjs/express#6017 (comment)](https://github.com/expressjs/express/issues/6017)

## Decision

We will adopt a policy where the caret (`^`) symbol is used for dependencies all dependencies.

**What will be done?**  

- Update the `package.json` files to use `^` for all dependencies (both prod and dev).

**Note on `^` vs. `~`:**  

- `^` allows updates to the most recent minor or patch version, offering greater flexibility and reducing the need for frequent manual updates. For example, `^1.2.3` will accept updates to `1.3.0`, `1.4.0`, but not `2.0.0`.
- `~` is more conservative, only allowing updates to patch versions. For example, `~1.2.3` will accept updates to `1.2.4`, `1.2.5`, but not `1.3.0`. Some Node.js legacy versions only support this, like Node@0.8.

**What will not be done?**  

- We won't force to use `^` or `~` for any dependency if there is a reason to use a pinned version and it is properly documented.

## Rationale

**Alternatives Considered:**

- **Alternative:** Continue pinning all dependencies, including internal ones, to specific versions.  
- **Reason for rejection:** This approach requires frequent updates and increases the maintenance burden, as each minor or patch update requires a new release.

**Pros and Cons**:

**Pros**:  

- Reduces the number of PRs for updating our own dependencies.
- Allows for quicker adoption of minor and patch updates within the Express ecosystem.
- Users are still protected by lockfiles, mitigating the risk of regressions.

**Cons**:  

- There is still a risk of minor regressions from updates, even within internally managed dependencies.
- Requires discipline in maintaining lockfiles to ensure stability for end users.

**Why is this decision the best option?**  
This decision strikes a balance between reducing maintenance effort and managing risk. It allows Express to leverage the benefits of semver for our own dependencies while maintaining control over external libraries that could introduce breaking changes.

## Consequences

**Positive Impact**:  

- Fewer manual updates required for internally managed dependencies.
- Users benefit from improvements and fixes in internal packages more quickly.

**Negative Impact**:  

- Potential risk of regressions if an internal dependency introduces a breaking change in a minor update.
- Users relying on strict version control may need to adjust their expectations when using our own dependencies.

**Mitigations**:

- Strong test coverage and CI checks will help catch potential issues early.
- Clear communication in documentation and release notes to inform users of the updated dependency policy.
- Ensure that we are following strict semver when releasing our own libraries.

## Implementation

- **Phase 1**: Update `package.json` files across all packages to apply `^` for all dependencies which support greater than node@0.10.0.
- **Phase 2**: Review and adjust documentation to include the new policy on dependency versioning.

## References

- [NPM semver documentation](https://docs.npmjs.com/cli/v6/using-npm/semver)

## Changelog

- **[2024-10-22]**: @UlisesGascon - Initial draft of ADR for using caret (`^`) or tilde (`~`) with our own dependencies.
