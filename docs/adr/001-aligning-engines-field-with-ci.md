# ADR 001: Aligning `engines` Field in `package.json` with CI Version Support

## Status
Proposed

## Submitters
- Ulises Gascón (@UlisesGascon)

## Decision Owners
- Express TC (@expressjs/express-tc)

## Context
The issue of maintaining consistency between the `engines` field in `package.json` and the versions tested in CI has been raised. Several packages have dropped support for Node.js versions prior to v18, but this change has not been consistently applied across all packages.

**Why do we need this decision?**  
Aligning the `engines` field with CI ensures that we explicitly define the minimum Node.js version supported for each release. This minimizes the risk of unintentional support for older versions that are not tested in CI, thus reducing maintenance overhead and unexpected bug reports.

**What problem does it solve or avoid?**  
This decision will help avoid introducing breaking changes unintentionally, streamline user expectations, and ensure that end users are aware of the minimum Node.js version they should use.

**Are there any existing issues/discussions/pull requests related to this?**  
- [Discussion: Using engines in the package.json #286](https://github.com/expressjs/discussions/issues/286)
- [Missing engines Field Update for Node.js Version Support in v2.0.0 pillarjs/finalhandler#64](https://github.com/pillarjs/finalhandler/issues/64)

## Decision
We will establish a policy that requires the `engines` field in `package.json` to match the Node.js versions defined in our CI configuration. This policy will apply to all packages within the Express ecosystem.

**What will be done?**  
- Update the `engines` field in `package.json` for each package to reflect the minimum Node.js version supported by the corresponding CI configuration.

**What will not be done?**  
- We will not support versions of Node.js that are not explicitly defined in the `engines` field or tested in CI.

## Rationale

**Alternatives Considered:**
- **Alternative 1:** Keep the `engines` field as-is and update only on a need basis.  
  - **Reason for rejection:** This could lead to confusion among users and contributors regarding the supported Node.js versions, resulting in unintended support for older versions.
- **Alternative 2:** Remove the `engines` field altogether and rely solely on documentation for version support.  
  - **Reason for rejection:** The `engines` field provides a built-in way to notify users of version requirements, making it an effective tool for communicating support boundaries.

**Pros and Cons**:
- **Pros**:  
  - Ensures clarity for users regarding the supported Node.js versions.
  - Reduces the risk of unintended support and compatibility issues.
  - Aligns with best practices for maintaining Node.js packages.
- **Cons**:  
  - Adjusting the `engines` field could be considered a breaking change, potentially impacting users on unsupported versions.

**Why is this decision the best option?**  
This decision creates a clear alignment between our CI-tested versions and the `engines` field, providing transparency for users and consistency across the project. It balances the need for modern support with a clear boundary for legacy versions.

## Consequences

**Positive Impact**:  
- Users will have a clear understanding of the supported Node.js versions, reducing confusion and unexpected bug reports.
- It allows the team to focus development on supported versions, leading to a more efficient development process.

**Negative Impact**:  
- Users on older Node.js versions may be forced to upgrade sooner than expected.

**Mitigations**:  
- Communicate this policy change clearly in release notes and documentation.

## References
- [NPM Documentation about `engines` field](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#engines)

## Changelog
- **[2024-10-22]**: @UlisesGascon - Initial draft of ADR for aligning `engines` with CI versions.
