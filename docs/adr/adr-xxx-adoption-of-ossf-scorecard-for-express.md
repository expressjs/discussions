# ADR XXX: Adoption of OSSF Scorecard for Express

## Status
Proposed

## Submitters
- @ulisesgascon
- @carpasse
- @inigomarquinez

## Decision Owners
- @expressjs/security-wg
- @expressjs/tc

## Context
The Open Source Security Foundation (OSSF) Scorecards project generates an automated "security score" for open source projects. This score aids users in assessing the security posture, risk level, and trustworthiness of a project, providing a standardized measure for comparing projects and making informed decisions when adopting new open-source dependencies. The scorecards can also facilitate automated decision-making processes for organizations, as new dependencies can be flagged for further evaluation if they fall below a specified security score threshold. This approach reduces the risk of malicious dependencies infiltrating production systems—a risk highlighted by recent incidents involving malicious NPM packages.

The Node.js organization has successfully implemented OSSF Scorecard monitoring, leading to notable security improvements across several repositories. Following this model, we propose adopting a similar approach for the Express framework. Tools like the OpenSSF Scorecard Monitor and Visualizer, along with established processes, make ongoing score management manageable and effective, providing straightforward ways to monitor and improve project security scores.

## Decision
The Express project will adopt OSSF Scorecard reporting as part of its security assessment and improvement practices.

### Actions
- Integrate OSSF Scorecards and establish monitoring through GitHub Actions with the OpenSSF Scorecard Monitor.
- Track actionable items highlighted by the Scorecard in individual PRs, which will detail specific security improvements.
- Engage contributors, including new collaborators, by involving them in the creation and review of Scorecard-related PRs.

### Exclusions
- We will not utilize the Step-Security auto-suggestion feature for PRs at this time, opting instead for manually curated and reviewed PRs. This will allow the security team to gradually onboard contributors and assess each change carefully.

## Rationale
The decision to adopt OSSF Scorecards stems from its demonstrated impact in similar environments, such as the Node.js project, where it has proven valuable for continuous security improvement and community engagement. Key considerations include:

- **Alternatives Considered:**
  - **Manual Security Audits**: Resource-intensive and lacks the automated frequency and granularity provided by Scorecards.

- **Pros and Cons**:
  - **Pros**: Provides automated, actionable insights; strengthens security posture; enables community involvement in a structured way; widely recognized within open source communities.
  - **Cons**: Initial setup and monitoring require dedicated resources; minor learning curve for contributors unfamiliar with the process.

The OSSF Scorecard is a mature, well-supported solution that aligns with Express's commitment to security and community engagement, making it the most fitting choice.

## Consequences
- **Positive Impact**: The OSSF Scorecard will contribute to Express's security posture by providing clear, actionable insights and facilitating ongoing improvement. It will also streamline the involvement of new collaborators, providing a welcoming entry point into Express contributions.
- **Negative Impact**: The Scorecard’s regular updates may require ongoing maintenance, and individual PR reviews could increase workload initially. Additionally, the Express organization has a large number of repositories, meaning each will require separate Scorecard implementation and upkeep until a centralized solution is found, increasing the management burden.
- **Mitigations**: Regular reviews in Security WG meetings, ongoing monitoring of scoring trends, and continued engagement with the triage team will help manage these challenges. We are also exploring the possibility of a centralized tool to streamline OSSF Scorecard implementation across all repositories, which could significantly reduce maintenance efforts.

## Implementation

- Already implemented in the 3 GitHub organizations related to Express ecosystem: [expressjs](https://github.com/expressjs), [pillarjs](https://github.com/pillarjs) and [jshttp](https://github.com/jshttp).

## References

- [OSSF Scorecards documentation](https://securityscorecards.dev/)
- [OpenSSF Scorecards project announcement](https://openssf.org/blog/2020/11/06/security-scorecards-for-open-source-projects/)
- [PR to add support for OSSF scorecard reporting in Express](https://github.com/expressjs/express/pull/5431)
- [PR to add OSSF scorecard in Node.js](https://github.com/nodejs/security-wg/issues/851)
- [Scorecards API for results](https://api.securityscorecards.dev/#/results)

## Changelog
- **[2024-10-30]**: @inigomarquinez - Drafted and proposed ADR for the adoption of OSSF Scorecard for Express.
