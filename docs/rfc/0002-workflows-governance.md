# Naming and Governance for the Central Repository of Reusable GitHub Actions

## Summary

This RFC proposes selecting an official repository name for hosting all shared and reusable GitHub Actions workflows used across the Express.js organization.

This RFC also proposes the creation of a new **Infrastructure Working Group (WG)** responsible for governing CI/CD standards, maintaining shared workflows, and ensuring consistent automation practices across all Express.js repositories.

## Motivation

To support recent RFCs around unified CI/CD and shared publishing workflows, the Express.js organization needs a **central, authoritative repository** for reusable GitHub Actions.

Today, no such repository exists, which leads to:

- Inconsistent naming conventions
- Repetition of similar workflows across repositories
- Fragmentation of CI/CD standards
- No clear ownership for shared automation tooling
- Difficulty onboarding contributors and maintainers

By choosing a clear, purpose-driven repository name with **`ci-workflows` proposed as the default**, we strengthen discoverability, consistency, and long-term governance.

Additionally, establishing an **Infrastructure WG** provides ongoing stewardship to ensure maintainers do not drift into divergent workflow configurations.

## Detailed Explanation

### Role of the Central "CI Workflows" Repository

The repository will host:

- Reusable workflows under `.github/workflows/*`
- Reusable actions under `.github/actions/*`
- Documentation and usage examples
- Versioned workflow releases (`v1`, `v2`, …)
- Organization-wide guidance for CI, publishing, and automation standards

### Role of the Infrastructure Working Group (WG)

The Infrastructure WG will be responsible for:

- Developing shared CI/CD standards
- Maintaining the central workflows repository
- Reviewing proposals and PRs for CI-related changes
- Managing security concerns (tokens, least-privilege permissions)
- Ensuring alignment with Node.js LTS support policies

This WG would include maintainers and contributors who actively work on CI/CD, release engineering, or build tooling.

## Naming Options

Below are several naming options, with **`ci-workflows`** recommended as the leading candidate.

### Option A — `expressjs/ci-workflows` (Recommended)

**Pros:**

- Very clear purpose: CI + reusable workflows
- Short, memorable, and easy to type
- Emphasizes the workflow-first focus
- Extensible enough to include publish workflows and automation
- Consistent with naming used by many OSS projects

**Cons:**

- Does not explicitly reference "actions"

### Option B — `expressjs/workflows`

**Pros:**

- Broad and flexible
- May include templates, actions, workflows, and automation

**Cons:**

- Less explicit about CI/CD purpose

### Option C — `expressjs/actions-workflows`

**Pros:**

- Extremely explicit: includes both actions and workflows
- Easy for contributors to identify purpose

**Cons:**

- Long and slightly redundant name

### Option D — `expressjs/infra-workflows`

**Pros:**

- Aligns with the concept of an Infrastructure WG
- Suggests broader automation responsibility

**Cons:**

- “Infra” may be interpreted as runtime or hosting infrastructure

### Option E — `expressjs/.github`

**Pros:**

- Standard GitHub convention followed by Node.js and others
- Automatically supports global templates

**Cons:**

- Less discoverable for CI/CD-specific workflows
- Repository may become cluttered over time

## Rationale and Alternatives

- A consistent naming standard improves visibility and contributor onboarding
- One repository prevents fragmentation of workflow logic
- A dedicated WG provides clarity, governance, and sustainability

Alternatives were considered but either lacked clarity, were overly broad, or did not express the CI/CD focus as well as `ci-workflows`.

## Implementation

### Step 1 — Approve Repository Name

Consensus among maintainers is required before creation.

### Step 2 — Create the Repository

The repository should include:

- README explaining purpose
- Contribution guidelines
- Actions and workflows directories
- Versioning strategy for reusable workflows

### Step 3 — Establish the Infrastructure WG

Define:

- Charter and responsibilities
- Membership and contribution rules
- Review and approval processes

### Step 4 — Adopt and Migrate

- Move reusable workflows to `ci-workflows`
- Update repositories to reference shared workflows
- Document usage in CONTRIBUTING.md

## Prior Art

- Multiple OSS projects maintain central workflow repositories
- GitHub endorses reusable workflows for multi-repo consistency
- Large OSS ecosystems often use a CI-dedicated repository for clarity and governance

## Unresolved Questions

N/A
