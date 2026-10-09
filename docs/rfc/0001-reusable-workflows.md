# Adopting shared & reusable GitHub Actions for publishing pipelines

## Summary

This RFC proposes standardizing all **publish and release automation** workflows across the Express.js organization using **shared, reusable GitHub Actions**. These workflows will handle packaging and secure publication to npm, ensuring consistent and reliable release processes across all repositories. This workflows will be centrally maintained, versioned, and consumed by individual repositories to ensure consistency, reliability, and easier maintenance.

## Motivation

Currently, each repository in the Express.js organization maintains its own GitHub Actions workflows. This leads to:

- Inconsistent CI/CD practices (different Node versions, test commands, caching, verification steps, etc.)
- Duplicated effort across repositories
- Increased probability of misconfiguration or security vulnerabilities
- Difficulty onboarding new contributors and maintainers
- Harder organization-wide upgrades (e.g., Node.js version updates or introducing new security validations)

By moving to shared reusable workflows, we expect to:

- Ensure consistent release standards across all repositories
- Reduce maintenance overhead and simplify updates
- Improve security by enforcing centralized best practices
- Enable easier shared improvements for all repositories
- Support future capabilities such as provenance and trusted publisher

## Detailed Explanation

### Proposal

- Create and maintain centralized reusable workflows under a repository such as:
  - `expressjs/ci-workflows` (repository name to be decided later)

- This repository will include:
  - `release.yml` — publish to npm and create GitHub releases (optional approval gates)

  ```yaml
  - uses: step-security/wait-for-secrets@v1
    id: wait-for-secrets
    with:
      secrets: |
        OTP: 
          name: 'OTP to publish package'
          description: 'NPM 2FA'

  - name: publish
    run: |
      export NODE_AUTH_TOKEN=${{ secrets.NPM_PUBLISH }}
      npm publish --otp ${{ steps.wait-for-secrets.outputs.OTP }} --access public
  ```

- Each repository will consume them using `workflow_call`, for example:

  ```yaml
  name: Publish package
  on:
    release:
      types: [released]

  permissions:
    id-token: write
    contents: read

  jobs:
    publish:
      uses: expressjs/ci-workflows/.github/workflows/release.yaml@v1
      secrets:
        NPM_PUBLISH: ${{ secrets.NPM_PUBLISH }} 
  ```

- Customization will still be possible using workflow inputs and conditionals.
- Workflows will be versioned (`v1`, `v1.1`, etc.) to ensure stability and exact sha can be used.

### Migration

1. Create shared workflow repository and initial pipeline definitions.
2. Document how to consume workflows, expected defaults, and available inputs.
3. Migrate a few core repositories (`express`, `router`, `body-parser`) as a pilot.
4. Expand adoption across the organization.
5. Deprecate custom pipelines once migration is completed.

## Rationale and Alternatives

| Approach | Pros | Cons |
|----------|------|------|
| **Shared reusable workflows (proposed)** | Consistent CI/CD, easier maintenance, improved security, one update applies to all repos | Requires initial setup and governance |
| **Status quo: per-repository workflows** | Maximum flexibility per repository | Inconsistent behavior, duplicated code, high maintenance, increased risk of outdated CI |

This proposal offers the best balance of maintainability, consistency, and openness for the Express.js ecosystem.

## Implementation

### Affected Areas

- All active repositories using GitHub Actions under the `expressjs/` organization.
- All active repositories using GitHub Actions under the `pillarjs/` organization.
- All active repositories using GitHub Actions under the `jshttp/` organization.
- No runtime code changes required; only CI configuration updates.

### Actions Required

- Create new organization repository.
- Implement and document reusable workflows.
- Configure organization-wide secrets (npm token, GitHub token, etc.) if required.
- Roll out reusable workflows in prioritized repositories.
- Establish contribution guidelines and versioning strategy for workflow updates.

### Technical Considerations

- Use `workflow_call` and workflow permissions properly.
- Use only organization-level secrets, not per-repo secrets for shared steps.
- Release workflows should include safeguards (e.g., manual approval for npm publish).

## Prior Art

- GitHub officially recommends this pattern for large organizations.

## Unresolved Questions and Bikeshedding

- What should the shared workflow repository be named?
- Should `npm audit` or CodeQL scanning be mandatory by default?
- How should versions of workflows be managed - tagged releases (`v1`) or branch references (`main`)?
- Should automatic npm releases be allowed, or manual approvals required?
- Do we want to pass the npm token using `export NODE_AUTH_TOKEN=${{ secrets.NPM_PUBLISH }}` or use the `.npmrc` file for that?
