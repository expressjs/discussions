# ADR 1396308b-0814-47bd-b8fc-6b3965d31295: GitHub Actions - to pin or not to pin

## Status

Proposed <!-- [Proposed | Accepted | Deprecated] -->

## Submitters

- @ctcpip

## Decision Owners

- @expressjs/express-tc

## Context

GitHub Actions support referencing actions via:

- Semantic versions (e.g., `actions/setup-node@v4`)
- Specific commit hashes (e.g., `actions/setup-node@49933ea`)
- Branch references (e.g., `actions/setup-node@main`)

Tools like [OpenSSF Scorecard](https://github.com/ossf/scorecard) recommend pinning to commit hashes to guarantee supply chain integrity and reduce the risk of malicious updates. However, GitHub's own documentation states:

> "All options are a tradeoff between guaranteed supply chain integrity and auto-patching of vulnerabilities in dependencies."

Therefore, there is no perfect solution; only pros and cons depending on which choice is made.

A recent discussion raised valid concerns about the real-world security value of pinning, particularly:

- **Pinning disables automatic updates**, including security patches, unless maintained manually.
- **Commit hashes are opaque**, harder to audit, and easy to misuse (e.g., referencing a malicious fork).
- **Most CI workflows in our repos don't access secrets**, minimizing risk even in the event of compromise.

Additionally, human verification of pinned commits, especially across many repos, can be error-prone or inconsistent.

## Decision

We will **not require pinning GitHub Actions to commit hashes** across all repositories.

Instead:

- Use **semantic version tags** (e.g., `@v4`) by default.
- Do **not use `@main` or other floating branch refs**.
- Repositories should **enable Dependabot for GitHub Actions** and set `schedule/interval` to a reasonable value, such as `monthly`.
- For workflows that **access secrets, publish artifacts, or have privileged scopes**, repository maintainers **may** choose to pin to commit hashes **if justified**.
- Where pinning is used:
  - A reviewer **must manually verify** the commit hash belongs to the correct source repository.
  - The verification step **must be noted in the PR review** to ensure accountability.

## Consequences

- **Improved maintainability**: Semantic versions reduce the burden of keeping commit hashes up to date.
- **Timely security updates**: Using version tags with Dependabot allows faster adoption of upstream patches.
- **Acknowledged risk**: While this approach reduces supply chain guarantees, the practical risk profile of most of our workflows does not justify the burden of pinning.
- **Flexibility for maintainers**: Projects/workflows with higher sensitivity can still opt into pinning where warranted.

## References

[Keeping your GitHub Actions and workflows secure](https://securitylab.github.com/resources/github-actions-building-blocks/)
[Mitigating Attack Vectors in GitHub Workflows](https://openssf.org/blog/2024/08/12/mitigating-attack-vectors-in-github-workflows/)
