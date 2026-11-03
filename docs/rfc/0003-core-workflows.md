# Standard default CI/CD workflows for Express.js repositories

## Summary

This RFC proposes a standardized set of default GitHub Actions workflows that should exist in every repository within the Express.js organization. These workflows ensure consistent validation on code pushes, pull requests, and post-merge events, along with optional compatibility and maintenance checks. The goal is to guarantee consistent quality across projects while keeping workflows lightweight and maintainable.

## Motivation

Currently, each repository uses its own GitHub Actions setup, or in some cases none at all. This creates:

- Inconsistent behavior across repositories
- Missed regressions in default branches
- Repeated maintenance effort
- Difficulty for new contributors to understand expectations
- No consistent testing against supported Node.js versions

By introducing standard workflows, we ensure:

- Consistent validation across repositories
- Predictable contributor experience
- Continued support for Node.js LTS policy
- Easier future migration to shared workflows

## Detailed Explanation

### Required Default Workflows

| Workflow | Trigger | Required | Description |
|----------|---------|----------|-------------|
| **1. Push Validation** | `on: push` to non-default branches | ✅ Yes | Run tests, lint, and basic validation for fast feedback during development. |
| **2. Pull Request Validation** | `on: pull_request` to `main`, `master`, or version branches | ✅ Yes | Ensures code is tested before merging. May include stricter checks than push validation. |
| **3. Post-Merge / Default Branch Check** | `on: push` to `main` or release branches | ✅ Yes | Ensures the default branch remains passing after merge and runs on the official Node.js LTS version(s). |
| **4. Manual Multi-Node.js Test** | `workflow_dispatch` | ✅ Yes (for core repos) | Allows manual execution of tests against multiple Node.js versions (e.g., 20, 22, 24) before releases or LTS changes. |
| **5. Scheduled Compatibility Check** | `schedule:` (optional) | Optional | Runs periodically to test against latest dependencies or Node.js nightly builds. |
| **6. Security / Dependency Audit** | Manual or scheduled | Optional | Runs `npm audit`, dependency checks, or license compliance. |
| **7. Release Workflow** | Tag or manual trigger | Optional | Builds and publishes packages (only for repositories that publish to npm). |

### Guiding Principles

- Workflows should remain lightweight and fast.
- No mandatory CodeQL, coverage upload, or heavy tasks.
- All default workflows must be compatible with reusable workflows.
- Repositories may add extra workflows if needed but should not remove core workflows.

## Rationale and Alternatives

| Approach | Pros | Cons |
|----------|------|------|
| **Proposed: Minimal Standard Workflows** | Balanced, lightweight, ensures consistency | Requires migration across repositories |
| Keep current undefined setup | No effort required | Inconsistency, higher risk of breakage |
| Enforce strict CI (coverage, audit, CodeQL, etc.) | Maximum protection | Too heavy for small modules, discouraging contributions |

## Implementation

### Affected Repositories

- All repositories under the Express.js organization that use or should use CI.

### Steps

1. Approve this RFC.
2. Create example workflow templates in `.ci-workflows/workflows/`.
3. Apply to core repositories:
   - expressjs/express
   - expressjs/router
   - expressjs/body-parser
4. Introduce shared workflows once this standard is normalized.
5. Document requirements in `CONTRIBUTING.md`.

## Prior Art

- GitHub recommends reusable organization workflows for consistency and maintenance.

## Unresolved Questions

- Which Node.js versions must be tested by default? (e.g., 22 + 24 vs 18 + 20 + 22 + LTS)
- Should push validation run on all branches or exclude `main`?
- Naming convention for workflows (`ci.yml`, `test.yml`, `validate.yml`)?
- Should `npm audit` or dependency checks be mandatory or optional?
