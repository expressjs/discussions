# Funding Tiers and Sponsor Rewards

## Summary

Replace Express's current flat two-tier funding structure (Backer $5/mo, Sponsor $100/mo, both with no described rewards) with an eight-tier ladder — Backer, Supporter, Patron, Bronze, Silver, Gold, Platinum, Diamond — each carrying explicit, escalating recognition rewards (SPONSORS.md → README logo → expressjs.com → docs). Top tiers use limited slot counts to create scarcity, with a single-slot Diamond tier as the apex. The same tier structure is mirrored across the project's funding surfaces (OpenCollective and GitHub Sponsors) so sponsors can fund Express through whichever platform their organization prefers. This RFC complements (and does not modify) [ADR 299: Funding Use Guidelines](../adr/299-funding-guidelines.md), which governs how funds are spent; this RFC addresses how sponsors are recognized.

## Motivation

Express is the most-depended-on web framework in the Node.js ecosystem, with over **100 million weekly downloads** on npm. Despite that footprint, the project's funding surfaces are underdeveloped:

- Only **two recurring tiers** exist: Backer at $5/mo and Sponsor at $100/mo.
- Neither tier describes any reward, recognition, or benefit.
- There is no public sponsor list, no logo placement, and no differentiation by contribution level.

This creates two concrete problems:

1. **Corporate sponsors cannot justify their support internally.** Marketing, DevRel, and developer-experience teams who would happily back Express need legible recognition (a logo, a placement, a public list) to expense it. Today, Express offers nothing to point to. Peer projects do — and they attract more sponsors as a result.
2. **There is no path above the entry sponsor tier.** $100/mo is the entry sponsor price across nearly every comparable project. Express has no tier above it, so larger organizations who want to support the project at a higher level have nowhere to go.

The expected outcome is a more predictable level of recurring sponsor support that helps fund what ADR 299 already authorizes — TC member living expenses, travel, and equipment — without requiring per-event fundraising.

Concrete peer comparison (snapshot, see Prior Art for detail):

- **NestJS** has 5 tiers from $2/mo to $1,000/mo with escalating logo placement and several hundred active sponsors.
- **Vue.js** publishes a 5-tier corporate ladder on [vuejs.org/sponsor](https://vuejs.org/sponsor/) (Bronze $100 → Platinum $2,000) plus a single-slot **Global Special Sponsor** at custom price as the apex.
- **Astro** has 6 tiers up to $10,000/mo (Exclusive Sponsor) with custom partnerships.
- **Preact** uses scarcity (5 Gold slots only) to encourage sponsors to commit at the higher tier.

Express has the brand and scale to support a comparable structure.

## Detailed Explanation

### Proposed tier ladder

| Tier | Amount (USD/mo) | Rewards | Slot cap |
|---|---|---|---|
| **Backer** | $5 | Name listed in `SPONSORS.md` in the [`expressjs/express`](https://github.com/expressjs/express) repo | unlimited |
| **Supporter** | $15 | Name + supporter badge in `SPONSORS.md` | unlimited |
| **Patron** | $50 | Name + patron badge in `SPONSORS.md` + name on the expressjs.com sponsors page | unlimited |
| **Bronze Sponsor** | $100 | Small logo in `README.md` of `expressjs/express` + name on the expressjs.com sponsors page | unlimited |
| **Silver Sponsor** | $500 | Medium logo in `README.md` + logo on the expressjs.com sponsors page | unlimited |
| **Gold Sponsor** | $1,500 | Logo on the **expressjs.com homepage** + larger logo in `README.md` | **5 slots** |
| **Platinum Sponsor** | $5,000 | Logo on the expressjs.com homepage **and** docs site header + top placement in `README.md` | **3 slots** |
| **Diamond Sponsor** | $10,000 | Exclusive top placement on `README.md`, expressjs.com homepage, and docs site header. Dedicated **case study card** on `expressjs.com/sponsors` (see Diamond case study card below). Recognition as Express's principal sponsor on each funding surface. | **1 slot** |

A **Custom Donation** option (one-time or recurring, any amount) remains available for contributors who do not match a tier.

### Reward specifics

- **Logo size guidelines** (to be finalized, see Unresolved Questions):
  - Bronze: ~120px width
  - Silver: ~180px width
  - Gold: ~240px width
  - Platinum: ~320px width, with priority placement above other tiers
  - Diamond: ~400px width, exclusive top-of-block placement above all other tiers
- **Link target**: each logo links to a URL provided by the sponsor at signup, subject to the acceptable-use policy below.
- **Refresh cadence**: sponsor lists are reconciled **quarterly** by a TC delegate. Lapsed sponsors are removed at the next refresh.
- **Eligibility / acceptable-use**: the TC reserves the right to decline or remove any sponsor whose business is incompatible with the project's values (e.g., projects that violate the Code of Conduct, illegal activity in major jurisdictions, or that misrepresent affiliation with Express). A short policy will be published on each funding surface's landing page.

### Where each tier is rendered

Recognition surfaces are split deliberately between two artifacts to keep the published npm package clean while preserving high-traffic GitHub exposure for paying sponsors.

| Tier | `SPONSORS.md` (not published) | `README.md` (GitHub + website only, stripped at publish) | expressjs.com |
|---|---|---|---|
| Backer ($5) | Name | — | — |
| Supporter ($15) | Name + badge | — | — |
| Patron ($50) | Name + badge | — | Name on `/sponsors` |
| Bronze ($100) | — | Small logo | Logo on `/sponsors` |
| Silver ($500) | — | Medium logo | Logo on `/sponsors` |
| Gold ($1,500) | — | Larger logo | Logo on homepage |
| Platinum ($5,000) | — | Top placement, larger logo | Logo on homepage **and** docs site header |
| Diamond ($10,000) | — | Exclusive top-of-block, largest logo | Top placement on homepage, docs site header, and `/sponsors` + dedicated case study card |

Two principles drive this split:

1. **Tiers below $100 live in `SPONSORS.md`**, a separate file at the repo root that is **excluded from the published npm tarball**. This keeps the package small and keeps npmjs.com's rendered README clean for tiers that don't carry logos. (The current naming convention `BACKERS.md` is replaced by `SPONSORS.md` since the file now covers Backer, Supporter, and Patron tiers.)
2. **Tiers from $100 (Bronze) up live in `README.md`**, which gives sponsors GitHub-page exposure (the highest-traffic surface a corporate sponsor cares about). To prevent these logos from inflating the package on npmjs.com, the sponsor block is **stripped from the README at publish time** (see Implementation §2).

### README sponsor block: strip-at-publish mechanism

The sponsor block in `README.md` is delimited by HTML comment markers:

```markdown
<!-- sponsors:start -->
... logo grid for Bronze through Diamond ...
<!-- sponsors:end -->
```

A `prepublishOnly` script removes everything between these markers (and the markers themselves) from the published tarball, then a `postpublish` script restores the original file. The result:

- **GitHub repo page**: full sponsor block visible (the high-value surface).
- **expressjs.com README mirror** (if used): full sponsor block visible.
- **npmjs.com package page**: sponsor block removed, README stays focused on usage.
- **Installed `node_modules/express/README.md`**: no sponsor block.

The strip is implemented as a small shell or Node script committed to the repo (e.g., `scripts/strip-sponsors.js`) and wired into `package.json` lifecycle scripts. CI verifies that publish-time output does not contain the markers or any logo image references.

This pattern is used by several projects in the ecosystem to balance sponsor visibility against npm package cleanliness.

### Slot-cap mechanics

Gold, Platinum, and Diamond tiers are intentionally scarce, mirroring the pattern used by Preact (5 Gold slots). Slot counts: **Gold 5**, **Platinum 3**, **Diamond 1**.

- Slot counts are **global across funding surfaces** (e.g., a sponsor occupying the Diamond slot via GitHub Sponsors fills the same slot that would otherwise be available on OpenCollective). The TC delegate maintains the canonical slot ledger.
- When all slots in a tier are filled, each funding surface will display the tier as **"Sold out"** (or the platform's equivalent).
- A **waitlist** is maintained in FIFO order across surfaces. When a slot is freed (downgrade, cancellation, removal), the next waitlisted sponsor is notified and given 14 days to confirm.
- Sponsors may downgrade to a lower tier at any time at the next billing cycle and lose their slot.
- The single Diamond slot is a deliberate signal: there is at most one "principal sponsor" of Express at any given time. If the slot is taken, prospective Diamond sponsors are invited to fund at Platinum and join the waitlist.

### Diamond case study card

To make the Diamond tier qualitatively distinct from Platinum (rather than "Platinum with a slightly bigger logo"), Diamond sponsors receive a dedicated **case study card** on `expressjs.com/sponsors`. The card consists of:

- The sponsor's logo at Diamond size.
- A **~200-word self-written description** of how the sponsor uses Express (provided by the sponsor).
- A link to the sponsor's relevant engineering blog post or product page.
- An optional one-paragraph quote from the sponsor's CTO/VP of engineering.

The card is **content authored by the sponsor**, not by the TC. The TC's involvement is limited to:

- A light editorial review for accuracy of any technical claims about Express, code-of-conduct fit, and acceptable-use policy compliance.
- A discretionary right to decline content that does not fit (the same right the TC holds for any sponsor under the acceptable-use policy).

The TC explicitly does **not** co-author, draft, or write content for sponsors. This boundary is intentional: it gives Diamond a meaningful marketing surface (a permanent narrative placement on expressjs.com that corporate marketing teams can link to) without creating a writing or deliverable obligation on the TC, which would conflict with ADR 299's framing (see also Alternative E).

The card is refreshed on the same quarterly cadence as the rest of the sponsor surfaces. Sponsors may submit updated content at each refresh.

### Funding surfaces

The same eight-tier structure is applied across all funding surfaces the project uses:

- **OpenCollective** ([opencollective.com/express](https://opencollective.com/express)) — current primary surface; tiers are reconfigured to match this RFC.
- **GitHub Sponsors** — a parallel surface enabled with the same tier names, prices, and rewards. Many corporate sponsors prefer GitHub Sponsors for procurement reasons (existing GitHub billing relationship, simpler invoicing, no third-party platform onboarding).

Sponsors choose whichever surface fits their procurement workflow; the rewards are identical. Slot counts are tracked globally (see Slot-cap mechanics) so a Diamond sponsor on GitHub Sponsors occupies the same single slot that would otherwise be visible on OpenCollective.

The TC delegate is responsible for reconciling sponsor lists across both surfaces during the quarterly refresh.

### Relationship to ADR 299

[ADR 299](../adr/299-funding-guidelines.md) governs how Express's funds are used (TC member living expenses, travel, equipment). This RFC does not change those guidelines. It only defines how sponsors are recognized and supported. The two together form a closed loop: tiered rewards encourage sponsor support, ADR 299 governs how the TC may request distributions from those funds.

## Rationale and Alternatives

### Alternative A: Keep the status quo (2 flat tiers, no rewards)

Pros: no work required, no risk of bikeshedding logo sizes.

Cons: leaves significant sponsor support unrealized. Express's footprint should attract sponsorship at the level NestJS and Vue achieve, but cannot without legible rewards.

### Alternative B: Webpack-style minimal (Backer + one flat Sponsor tier at $200/mo)

Pros: simple to maintain, single sponsor list to manage.

Cons: gives larger sponsors no path above the entry tier. Webpack itself has plateaued in sponsor support under this model and relies heavily on its handful of large named sponsors. Does not give Express room to recognize organizations who want to support the project at a higher level.

### Alternative C: Keep all tiers in `README.md` (no strip-at-publish)

What Vue and NestJS do today.

Pros: simplest possible setup, no scripts, no CI check.

Cons: every sponsor logo bloats the README rendered on npmjs.com, where users browse for usage information. The published tarball also carries unnecessary sponsor markup. Over time, with a growing sponsor list, this gets visually noisy and increases package size marginally.

### Alternative D: Move all logos out of `README.md` (website-only)

Keep `README.md` clean of sponsor logos entirely; surface logos only on `expressjs.com/sponsors` and a non-published `SPONSORS.md`.

Pros: simplest README, no publish-time tooling, smallest tarball.

Cons: removes the highest-value reward — GitHub repo page exposure — which is what corporate sponsors care most about (it is the most visited and most archived surface). Likely reduces sponsor sign-ups at Bronze and above.

### Alternative E: Babel-style with paid support hours at top tier

Babel's top tier ($24,000/yr) bundles 2 hours of maintainer support time. This attracts the largest individual sponsor commitments.

Pros: highest per-sponsor commitment.

Cons: **rejected** — creates a service obligation on the TC that conflicts with ADR 299's framing (funds support contributors at their discretion, not pre-committed deliverables). Would also require a contract / SLA layer that the project does not currently have the capacity to maintain.

### Chosen approach: 8-tier ladder with logo rewards + slot scarcity

Best risk/reward balance:

- **Recognition-only rewards** keep the TC's obligations minimal — no support contracts, no deliverables.
- **Eight tiers** cover the full range from individual supporter ($5) to a single-slot Diamond apex ($10,000). Two intermediate individual-friendly tiers ($15, $50) close the 20× gap between Backer and Bronze, bridging the audience between individual supporter and corporate sponsors (prior art: Preact's Supporter tier at $20/mo). The Diamond tier provides a clear "principal sponsor" headline placement that a single committed company can claim.
- **Slot caps on Gold (5), Platinum (3), Diamond (1)** create a sense of meaningful recognition without overpromising; they cost the project nothing if unfilled and tend to attract sponsors at higher tiers more effectively than uncapped tiers. The single Diamond slot is the strongest scarcity signal available.
- **Cross-reference to ADR 299** keeps the "how sponsors are recognized" and "how funds are used" concerns separate and avoids re-litigating the funding guidelines.

## Implementation

Once accepted, implementation steps (in order) are:

1. **Reconfigure tiers on each funding surface** — apply the eight-tier structure to both [OpenCollective](https://opencollective.com/express) (replacing the existing two tiers) and GitHub Sponsors. Configure slot limits on Gold (5), Platinum (3), and Diamond (1) on every surface, with the global slot-ledger mechanism described under Slot-cap mechanics.
2. **Add `SPONSORS.md`** at the root of `expressjs/express` listing all Backer / Supporter / Patron sponsors with their badges. Add `SPONSORS.md` to the `files` whitelist exclusion (or `.npmignore`) so it is **not included in the published npm tarball**. Automate the sync from each funding surface (OpenCollective bot, GitHub Sponsors API, or a scheduled GitHub Action that pulls from both).
3. **Add a delimited sponsor block to `expressjs/express` README.md** showing Bronze / Silver / Gold / Platinum / Diamond logos with appropriate sizing, between `<!-- sponsors:start -->` and `<!-- sponsors:end -->` markers. Diamond appears at the top of the block when filled.
4. **Add a strip-at-publish script** (`scripts/strip-sponsors.js` or equivalent) and wire it into `prepublishOnly` (strips block from `README.md`) and `postpublish` (restores it). Add a CI check that runs the strip and asserts the published tarball contains no logo references or sponsor markers.
5. **Add a `/sponsors` page to expressjs.com** listing Patron through Diamond tiers with names/logos and links. Diamond, when filled, appears at the top of the page with its dedicated case study card (logo, ~200-word sponsor-authored description, link, optional CTO/VP quote).
6. **Add a logo strip to the docs site header** showing Platinum and Diamond sponsors.
7. **Publish an acceptable-use policy** on each funding surface's landing page describing eligibility, removal criteria, and the quarterly refresh cadence.
8. **Designate a TC delegate** as the owner of the quarterly cross-surface reconciliation pass, the global slot ledger, and the waitlist.
9. **Cross-link** to ADR 299 from each funding surface's landing page so prospective sponsors understand how funds are used.

Affected repositories / sub-components:

- `expressjs/express` — `README.md`, new `SPONSORS.md`, new strip-at-publish script, `package.json` lifecycle scripts, `.npmignore` or `files` field
- `expressjs/expressjs.com` — homepage, new `/sponsors` page
- Express docs site — header logo strip
- OpenCollective and GitHub Sponsors settings (no code change, configuration only)

The only code changes to Express itself are the strip-at-publish script and the corresponding `package.json` lifecycle hooks; the runtime is unaffected.

## Prior Art

The following large open-source projects all use multi-tier funding structures with explicit rewards. Amounts are in USD per month unless noted. Most use OpenCollective; some (Vue.js) supplement with their own sponsorship pages and use both OpenCollective and GitHub Sponsors as funding surfaces.

| Project | Lowest tier | Sponsor entry | Mid-tier | Top tier | Reward style |
|---|---|---|---|---|---|
| **NestJS** | Backer $2 | Bronze $100 | Silver $250 / Gold $500 | Base $1,000 | README + homepage + docs (escalating placement) |
| **Astro** | Backer $10 | Sponsor $100 | Gold $250 / Theme $200 | Platinum $1k / Exclusive $10k | README → homepage → docs → custom partnership |
| **Vue.js** | Individual Backer $5 | Bronze $100 | Silver $250 / Gold $500 | Platinum $2k + **Global Special Sponsor** (1 slot, custom-priced, currently vacant) | SPONSORS.md → README + homepage → docs sidebar (Platinum) → exclusive above-the-fold (Special Sponsor) |
| **Babel** | Backer $2 | Bronze $100 | Silver $500 / Gold $1k | Base Support $24k/yr | Logo + maintainer support hours at top |
| **Preact** | Backer $2 + Supporter $20 | Bronze $100 | Gold $500 (5 slots) | Platinum $1k | README logo + scarcity (limited slots) |
| **Fastify** | Tier 1 $5 | Tier 3 $100 | Tier 4 $300 | — | SPONSORS list placement |
| **Webpack** | Backer $2 | Sponsor $200 | — | — | Flat, recognition only |

Notable patterns:

- **$100/mo** is the near-universal entry sponsor price.
- **5-tier ladders** dominate among peers (Backer → Bronze → Silver → Gold → Platinum / Diamond). Express's proposed 8 tiers is intentionally denser at the individual end (adding Supporter and Patron) and at the apex (adding a single-slot Diamond), based on Express's larger footprint and the gaps the 5-tier model leaves between $5 and $100.
- **Reward escalation**: nothing → SPONSORS.md → README logo → website logo → homepage hero → docs placement → custom partnership.
- **Preact's slot scarcity** appears to be the most effective mechanism for encouraging sponsors to commit at higher tiers.
- **curl** does not use OpenCollective; it relies on GitHub Sponsors and a corporate employer relationship (wolfSSL employs Daniel Stenberg). Not a useful direct comparison for our model.

## Unresolved Questions and Bikeshedding

- **Exact dollar amounts**: $5 / $15 / $50 / $100 / $500 / $1,500 / $5,000 / $10,000 are anchored against peers but the TC may want to adjust. In particular, is Diamond at $10,000/mo realistic for Express to attract, or should it sit closer to $7,500? And do the Supporter/Patron tiers ($15, $50) sit at the right values to encourage individual supporters to choose them?
- **Slot-cap counts**: 5 Gold / 3 Platinum / 1 Diamond are starting points. Should these scale with demand, or stay fixed to preserve scarcity value?
- **Tier names**: Bronze / Silver / Gold / Platinum / Diamond is the dominant convention (Vue.js uses Diamond at the apex) but somewhat impersonal. Alternatives include thematic names (Fastify uses Tier 1–4, Astro uses descriptive names) and non-metal apex names (e.g., "Principal Sponsor", Iridium, Titanium). Worth a quick discussion.
- **Diamond case study card editorial process**: who on the TC owns the light review pass? What's the SLA on reviewing submitted content (e.g., 14 days)? Is there a published rejection-criteria list, or is it left to TC discretion?
- **Logo size pixel values**: 120 / 180 / 240 / 320 / 400 px widths are illustrative. Final values should be set by whoever owns expressjs.com design.
- **Acceptable-use policy wording**: the policy needs to be written. Should it follow an existing model (e.g., the OpenJS Foundation's sponsorship policy)?
- **Theme-style content-tied tier**: Astro has a "Theme Sponsor" tier ($200/mo) that promotes themes in their catalogue. Express has no equivalent content surface today, but a future "Showcase Sponsor" or "Tutorial Sponsor" tier could be considered if a corresponding catalogue is built.
- **Sync automation between funding surfaces**: should the OpenCollective ↔ GitHub Sponsors reconciliation be fully automated (a scheduled job that pulls from both APIs and produces a unified `SPONSORS.md` and logo manifest), or kept manual on the quarterly cadence? Automation reduces TC overhead but adds a maintenance burden if APIs change.

{{THIS SECTION SHOULD BE REMOVED BEFORE RATIFICATION}}
