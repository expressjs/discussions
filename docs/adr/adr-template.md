# ADR template

This is the base template that we use

```md

# ADR [Number]: [Title of Decision]

## Status

[Proposed | Accepted | Deprecated | Superseded by ADR [number]]

## Submitters

[List of people who proposed this decision. Include GitHub usernames or names with contact information if needed.]

- @username1
- @username2

## Decision Owners

[List of people responsible for driving the decision and following up on its implementation. This may include teams or subject matter experts.]

- @owner1
- @team1

## Context

Describe the problem, need, or feature request that prompted this decision. Include any relevant background information, constraints, and considerations that were taken into account.

- **Why do we need this decision?**
- **What problem does it solve or avoid?**
- **Are there any existing issues/discussions/pull requests related to this?** (Include links to relevant GitHub issues, forum threads, or discussion channels.)

## Decision

Clearly state the decision that was made. Describe the chosen solution or approach in detail so that others can understand what was decided.

- **What will be done?**
- **What will not be done?** (If applicable, specify what was explicitly ruled out.)

## Rationale

Explain why this decision was made, including a discussion of the alternatives considered and why they were not chosen.

- **Alternatives Considered:**
  - Alternative 1: [Description and reasons for rejection]
  - Alternative 2: [Description and reasons for rejection]
- **Pros and Cons**: Outline the pros and cons of the chosen solution.
- **Why is this decision the best option?** (Explain the key factors that influenced this choice.)

## Consequences

Describe the positive and negative outcomes of the decision, including any potential risks or technical debt.

- **Positive Impact**: What benefits does this decision bring to the project?
- **Negative Impact**: What challenges or limitations does this introduce?
- **Mitigations**: How will we address potential drawbacks or issues?

## Implementation

(Optional, if relevant)
Outline the steps required to implement the decision. This section is particularly useful if the decision involves a series of actions or a roadmap.

- **Phase 1**: [Description]
- **Phase 2**: [Description]
- **Estimated Effort**: Provide a rough estimate of time or effort needed.

## References

Include any external links, documents, discussions, or research that were referenced during the decision-making process.

- [Link to relevant GitHub issue or pull request](#)
- [Link to forum discussion](#)
- [Documentation or research sources](#)

## Changelog

Track changes or updates to this ADR over time. Include the date, author, and a brief description of each change.

- **[YYYY-MM-DD]**: [@username] - [Brief description of the change]
  - Example: **[2024-10-22]**: @owner1 - Updated the decision to include support for Redis caching.
- **[YYYY-MM-DD]**: [@username] - [Brief description of the change]
  - Example: **[2025-01-15]**: @username2 - Deprecated this ADR due to a shift in the database strategy.

```
