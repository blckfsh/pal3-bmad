---
stepsCompleted:
  - "step-01-document-discovery"
  - "step-02-prd-analysis"
  - "step-03-epic-coverage-validation"
  - "step-04-ux-alignment"
  - "step-05-epic-quality-review"
  - "step-06-final-assessment"
readinessStatus: "READY"
readinessStatusHistory: "NEEDS WORK at assessment; READY after two remediation passes on 2026-08-12"
issuesFound: 20
filesAssessed:
  - "_bmad-output/planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md"
  - "_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/DESIGN.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/EXPERIENCE.md"
  - "_bmad-output/planning-artifacts/epics.md"
---

# Implementation Readiness Assessment Report

**Date:** 2026-08-12
**Project:** paluwagan3

## Document Inventory

### PRD Files Found

**Whole Documents:**
- `prds/prd-paluwagan3-2026-08-08/prd.md` (40,437 bytes, modified 2026-08-08) — status `final`

**Sharded Documents:**
- None

### Architecture Files Found

**Whole Documents:**
- `architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md` (17,508 bytes, modified 2026-08-08) — status `final`
- `architecture/architecture-paluwagan3-2026-08-08/SCF-architecture-outline.md` (11,643 bytes, modified 2026-08-08) — grant-facing narrative, not the build substrate

**Sharded Documents:**
- None

### Epics & Stories Files Found

**Whole Documents:**
- `epics.md` (103,117 bytes, modified 2026-08-12) — 5 epics, 44 stories

**Sharded Documents:**
- None

### UX Design Files Found

**Whole Documents:**
- `ux-designs/ux-paluwagan3-2026-08-11/DESIGN.md` (14,312 bytes, modified 2026-08-11) — status `final`, visual identity
- `ux-designs/ux-paluwagan3-2026-08-11/EXPERIENCE.md` (24,905 bytes, modified 2026-08-11) — status `final`, IA / behavior / states / flows

**Sharded Documents:**
- None

### Supporting Documents (not assessed as required types)

- `briefs/brief-paluwagan3-2026-08-08/brief.md` (17,241 bytes, 2026-08-08)
- `briefs/brief-paluwagan3-2026-08-08/addendum.md` (7,750 bytes, 2026-08-08)
- `ux-designs/ux-paluwagan3-2026-08-11/mockups/room-terms-commit.html` — composition reference cited by both UX spines

### Issues Found

**Duplicates (whole + sharded):** none. No document type exists in both forms, so there is no version ambiguity to resolve.

**Missing required documents:** none. PRD, Architecture, Epics & Stories, and UX are all present.

**Selection note — two architecture documents:** `ARCHITECTURE-SPINE.md` is the build substrate (15 architecture decisions, stack, source tree, capability map) and is the document assessed here. `SCF-architecture-outline.md` is a grant-facing narrative derived from it and is excluded from the assessment; it is not a competing version and needs no resolution.

**Project knowledge:** `project_knowledge` resolves to `docs/`, which is empty, and no `project-context.md` exists anywhere in the repository. Expected for a greenfield project with no code yet; noted because it means no coding-standards or codebase context is available to ground this assessment.

### Documents Selected for Assessment

| Type | File |
|---|---|
| PRD | `prds/prd-paluwagan3-2026-08-08/prd.md` |
| Architecture | `architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md` |
| UX | `ux-designs/ux-paluwagan3-2026-08-11/DESIGN.md` + `EXPERIENCE.md` (spine pair, assessed together) |
| Epics & Stories | `epics.md` |

## PRD Analysis

Source: `prds/prd-paluwagan3-2026-08-08/prd.md` (status `final`, whole document, read in full). Requirements are numbered FR-1…FR-23 in the PRD itself and grouped by feature area in §4. Each FR carries a set of testable "Consequences"; those consequences are the substance validated against story acceptance criteria in Step 3.

### Functional Requirements

**§4.1 Identity and Verification**

- **FR-1: Member identity verification.** A prospective Member can complete identity verification through the KYC Provider before joining any Room. Consequences: a wallet with no valid KYC Attestation cannot join and the join transaction reverts; Pal3 stores only attestation status, verification tier, issue date, and provider reference — no name, document image, employment record, or bank detail; an expired attestation blocks joining a *new* Room but does not remove a Member from a Room in progress.
- **FR-2: Verification tier sets starting Trust Score.** Consequences: basic verification (government ID only) yields 300 `[ASSUMPTION]`; full verification (ID + employment record + bank detail) yields 400 `[ASSUMPTION]`; a Member cannot hold a score above 400 until at least one Cycle is complete.
- **FR-3: One human, one identity.** Consequences: the KYC Provider's duplicate-detection result is enforced at attestation issue and a second attestation for the same individual is rejected; a join attempt from a wallet whose attestation is flagged duplicate reverts.

**§4.2 Room Lifecycle**

- **FR-4: Room creation.** An Underwriter can create a Room specifying Contribution amount, Cadence, and Member count. Consequences: Member count must be within the Underwriter's subscription tier capacity or the request reverts; creation requires posted Backstop meeting FR-19 before the Room accepts Members; V1 accepts weekly Cadence only, other values revert `[ASSUMPTION]`.
- **FR-5: Member admission.** Consequences: the Underwriter view exposes Trust Score, Cycles completed, Default count, and account age — never name, documents, employer, or bank details; a rejected applicant's locked funds are returned in full.
- **FR-6: Room start.** A Room starts automatically when it reaches its configured Member count, and only then. Consequences: Payout Positions computed and frozen at start; parameters immutable at start, with no Member added or removed except through Stake exhaustion and no amount, Cadence, or ordering change; failure to fill within the open window cancels the Room and returns every Stake and Contribution in full `[ASSUMPTION: 14-day window]`.
- **FR-7: Room completion.** Consequences: every Member who completed all Rounds has their full Stake returned; Trust Score cycle-completion credit applied at close.

**§4.3 Contributions and Escrow**

- **FR-8: Scheduled contribution.** Consequences: Contributions accepted only for the current open Round; escrowed funds withdrawable by no party under any code path other than Payout (FR-12), Slash (FR-15), or cancellation refund (FR-6); a Member who has already contributed for the current Round cannot double-pay.
- **FR-9: Contribution status visibility.** Consequences: status shows per Member — paid, pending, in Grace Window, or defaulted — identified by Member handle, not legal identity.

**§4.4 Trust Engine**

- **FR-10: Trust Score computation.** 0–1000 per Member, updated on defined events. Consequences: on-time Contribution +10; cured within Grace Window +2; Default −150; Cycle completed with zero Defaults +50; Room abandoned via Stake exhaustion −400 (all `[ASSUMPTION]`); clamped to [0, 1000] after every update; changes deterministic and reproducible from event history.
- **FR-11: Payout ordering with Fairness Floor.** Consequences: ranked by Trust Score descending with the highest score paid in Round 1; ties broken deterministically and auditably by a published rule with no randomness `[ASSUMPTION: earlier Room-join timestamp]`; a Member assigned the final position in the two immediately preceding Cycles within the same Room cannot hold it a third consecutive time and swaps with the next-lowest-ranked eligible Member; positions visible to every Member before start and frozen thereafter.
- **FR-12: Pot payout.** Consequences: recipient receives the full Pot — Member count × Contribution — with no platform deduction; Payout executes on the scheduled Round boundary regardless of Default events, having drawn on Slash and Backstop as needed; each Member receives exactly one Payout per Cycle.
- **FR-13: Stake sizing.** Consequences: Stake = Contribution × (2 − TrustScore/1000), yielding 2× at score 0 and 1× at score 1000 `[ASSUMPTION]`; displayed to the Member before commitment with the score that produced it; locked at join and not recomputed mid-Cycle even if the score changes.

**§4.5 Default Handling**

- **FR-14: Grace Window.** 48 hours after a missed deadline during which payment is accepted without a Default. Consequences: the Round does not advance and no Payout occurs while the window is open; the Member is notified at window open and at 24 hours remaining; a Contribution paid within the window is accepted in full and recorded as cured, applying the cured-contribution score change rather than a Default penalty; other Members see "awaiting contribution" with time remaining.
- **FR-15: Slash and Backstop waterfall.** Consequences: the Slash amount equals exactly the missed Contribution, not the whole Stake; if remaining Stake is less than the missed Contribution, the entire remaining Stake is slashed and the Backstop covers the difference; the scheduled recipient receives the full Pot in the same Round with no delay and no reduction in every branch; the defaulting Member's Trust Score takes the FR-10 Default penalty; the Round advances immediately on resolution.
- **FR-16: Stake exhaustion and removal.** Consequences: removal occurs at the moment a Slash leaves the Stake at zero; the Backstop covers that Member's Contributions for all remaining Rounds; a removed Member who has not yet received their Payout does not receive one, while one who already received it retains it and the Backstop absorbs the loss; the abandonment penalty applies.
- **FR-17: Underwriter exposure visibility.** Consequences: maximum exposure computable and displayed before the Underwriter commits to a Room; live exposure updates on every Slash and Backstop draw.

**§4.6 Underwriter Subscription and Capacity**

- **FR-18: Tiered capacity.** Consequences: a Room creation request exceeding concurrent-Room or Members-per-Room limits reverts; V1 operates a single tier — one Room, maximum 10 Members `[ASSUMPTION]`.
- **FR-19: Capital adequacy gate.** Consequences: required Backstop is a published function of Member count and Contribution amount `[ASSUMPTION: minimum 2 × Contribution × Member count]`; a Room cannot open Member admission until the required Backstop is posted and locked; Backstop cannot be withdrawn while any Room it collateralizes is in progress.
- **FR-20: Per-contribution fee.** Consequences: displayed as both a percentage and an absolute per-Round amount on the Room terms screen before a Member commits; charged on Contribution, not on Payout, so a Member's Pot is never reduced; the platform takes no share of the fee and no share of the Pot.

**§4.7 Member Application**

- **FR-21: Wallet connection.** Consequences: wallet connection implemented through WalletConnect rather than a browser-extension API, because Soroban contract invocation requires `signAuthEntry` and that path against Freighter Mobile must go through WalletConnect directly; at least one WalletConnect-compatible Stellar wallet supported end to end `[ASSUMPTION: Freighter via WalletConnect]`; a Member with insufficient balance for Stake plus first Contribution is told the exact shortfall before attempting to join.
- **FR-22: Room terms disclosure.** Consequences: one screen shows Contribution amount, Cadence, Member count, their Payout Position and date, their Stake, the Underwriter's fee, and the default-handling waterfall in plain language; the screen states explicitly that funds are denominated in a USD-pegged stablecoin and not in pesos and that the peso value may move over the Cycle; the Member must acknowledge the FX disclosure before joining, and the acknowledgement is recorded.
- **FR-23: Schedule and status view.** Consequences: shows all Rounds, all Payout Positions and dates, and which Rounds are complete; shows their own Trust Score and its change history.

**Total FRs: 23**

### Non-Functional Requirements

Drawn from §10 Cross-Cutting NFRs plus feature-specific and constraint statements in §4.1, §9, and §12. The PRD does not number these; the numbering below was assigned during epic creation and is used consistently in `epics.md`.

- **NFR-1: Funds safety is the dominant quality attribute.** Contracts holding escrow, Stake, and Backstop must pass an audit-readiness review before any mainnet Room opens with real value (brief milestone T2). No mainnet pilot begins without it.
- **NFR-2: Determinism.** Payout ordering, Slash amounts, and Trust Score changes must be reproducible from event history by any observer. No randomness anywhere in the system.
- **NFR-3: No privileged override.** No admin key, upgrade path, or operator action can move escrowed funds, alter a Payout Position mid-Cycle, or reverse a Slash. This constraint outranks operational convenience.
- **NFR-4: Availability at Round boundaries.** Contribution and Payout must succeed at scheduled boundaries; off-chain trust-service unavailability must never block an on-chain Payout.
- **NFR-5: Cost per Round** must stay negligible relative to a ₱1,000-equivalent Contribution; transaction cost is a product constraint, not just an engineering one.
- **NFR-6: KYC boundary (§4.1).** Identity document handling is out of Pal3's technical boundary by design; the KYC Provider is the sole personal information controller for document data.
- **NFR-7: No personal data on-chain (§9).** Wallet addresses, Trust Scores, and Room state are pseudonymous; the link to a legal identity exists only in the KYC Provider's records.
- **NFR-8: Irreversibility disclosure (§9).** Trust Score history is append-only and a Default is permanent; Members must be told before joining that this record cannot be erased.
- **NFR-9: Open source (§12).** All Soroban contracts published under a permissive licence from first deployment `[ASSUMPTION: Apache 2.0 over MIT, for the patent grant]`.
- **NFR-10: Surface constraint (§12).** Mobile-first web application; PWA rather than native for V1 to avoid app-store review inside the eight-week window `[ASSUMPTION]`.
- **NFR-11: Trust Engine portability (§12).** Computed off-chain as service code, deliberately chain-agnostic so the mechanism remains portable if Pal3 later migrates to reach a peso rail.

**Total NFRs: 11**

### Additional Requirements

**Scope boundaries (§5, §6.2) that constrain story scope:**
- Pal3 is not a lender, not a yield product, not a custodian, not an exchange or on/off-ramp.
- No dispute resolution in V1: no appeals process, no manual override, no admin key that can reverse a Slash.
- Out of MVP scope: peso-stablecoin rail, fiat on/off-ramp, open public enrolment, Underwriter subscription billing, multi-Room trust-graph effects, employer-sponsored rooms, multi-currency, secondary markets.

**Platform constraints (§12):** Stellar/Soroban with Rust/WASM contracts; USD-pegged stablecoin; Trust Engine computed off-chain and committed on-chain at consequence points; no verifiable-randomness dependency anywhere.

**Success metrics (§7) that imply verification work:** SM-1 one completed Cycle; SM-2 at least one Default resolved through the full waterfall, induced if it does not arise naturally; SM-3 zero loss to any scheduled recipient; SM-5 ≥80% of the cohort able to state their score and why it changed.

**Open questions (§13) with a pre-mainnet revisit condition:**
1. **Grace Window abuse** — FR-14 has no escalation on repeated cures; a Member could cure every Round, taking +2 rather than −150 and delaying every Payout by up to 48 hours. *Owner: founder. Revisit: before pilot recruitment.*
2. **Backstop adequacy** — verify the FR-19 formula against a full exposure table, including the Round-1 recipient defaulting immediately after Payout. *Owner: founder. Revisit: before posting Backstop.*
3. **Counsel confirmation** on the §8 regulatory posture and the §9 erasure-rights tension. *Owner: PH fintech counsel, milestone T3.*
4. **Named pilot cohort** — size settled at 5–10, individuals not identified. *Owner: founder. Revisit: during the build window.*

Items 5–9 (Underwriter lapse mid-Cycle, Trust Score decay, rehabilitation, cross-Room Fairness Floor, Sybil resistance beyond KYC) are deferred past V1 as structurally absent in a single-Room, single-Underwriter pilot.

### PRD Completeness Assessment

The PRD is unusually complete for this stage and is genuinely implementable. Strengths that matter for traceability:

- **Every FR carries explicit testable consequences.** This is the largest single contributor to downstream story quality — acceptance criteria could be derived from the PRD rather than invented.
- **Assumptions are tagged inline and indexed in §14**, so every soft number (score deltas, stake formula, backstop multiple, grace duration, open window) is visible as a decision rather than buried as a fact.
- **The falsifiable claim is named.** §4.4 states that if Defaults concentrate among high-trust, early-paid Members, the thesis is wrong and Stake sizing must be rebuilt against remaining obligation. A PRD that names what would disprove it materially reduces the risk of a pilot that proves nothing.
- **Open questions are triaged with owners and revisit conditions** rather than left as an undifferentiated list.

Gaps carried into the assessment, none of which block implementation:

- **NFRs are unnumbered in the PRD.** Numbering was assigned during epic creation, so NFR references in `epics.md` cannot be verified against the PRD by string match — only by reading. A traceability inconvenience rather than a defect.
- **NFR-5 has no threshold.** "Negligible relative to a ₱1,000-equivalent Contribution" is not a number. §12 supplies the reasoning (a $0.50 fee on ~$18 is close to 3%) but states no pass/fail line.
- **FR-14 specifies notification without a delivery channel.** The PRD requires notification at Grace Window open and at 24 hours remaining but does not say by what means. The UX spine later constrains V1 to in-app only and records the resulting reach gap.

## Epic Coverage Validation

Coverage was validated two ways: against the `FR Coverage Map` the epics document declares, and independently by extracting every `FR-n` citation from story acceptance criteria and mapping it back. Both agree. The table below reports the *acceptance-criteria* mapping, which is the stronger evidence — a coverage map can claim anything, but an FR cited inside a Given/When/Then is a testable commitment.

### Coverage Matrix

| FR | PRD requirement | Stories carrying it in acceptance criteria | Status |
|---|---|---|---|
| FR-1 | Member identity verification | 1.3, 1.5, 1.7, 1.8, 2.5 | ✓ Covered |
| FR-2 | Verification tier sets starting Trust Score | 1.4, 1.5, 1.7, 4.1 | ✓ Covered |
| FR-3 | One human, one identity | 1.3, 1.5, 1.7, 2.5 | ✓ Covered |
| FR-4 | Room creation | 2.1 | ✓ Covered |
| FR-5 | Member admission | 2.4, 2.5, 4.3 | ✓ Covered |
| FR-6 | Room start | 2.5, 2.6, 2.7, 2.10, 3.5 | ✓ Covered |
| FR-7 | Room completion | 3.6, 3.13, 4.4 | ✓ Covered |
| FR-8 | Scheduled contribution | 3.1, 3.9 | ✓ Covered |
| FR-9 | Contribution status visibility | 3.3, 3.10, 4.3 | ✓ Covered |
| FR-10 | Trust Score computation | 1.4, 4.1, 4.2, 4.4, 4.5 | ✓ Covered |
| FR-11 | Payout ordering with Fairness Floor | 2.6, 2.8, 2.10 | ✓ Covered |
| FR-12 | Pot payout | 3.2, 3.3, 3.4, 3.12 | ✓ Covered |
| FR-13 | Stake sizing | 2.5, 2.8, 4.2, 4.3 | ✓ Covered |
| FR-14 | Grace Window | 3.3, 3.10, 3.11 | ✓ Covered |
| FR-15 | Slash and Backstop waterfall | 3.4, 3.11, 3.12 | ✓ Covered |
| FR-16 | Stake exhaustion and removal | 2.8, 3.5, 3.12 | ✓ Covered |
| FR-17 | Underwriter exposure visibility | 2.3, 3.14 | ✓ Covered (split) |
| FR-18 | Tiered capacity | 2.1, 2.3 | ✓ Covered |
| FR-19 | Capital adequacy gate | 2.2, 2.3, 5.5 | ✓ Covered |
| FR-20 | Per-contribution fee | 2.8, 3.1, 3.6 | ✓ Covered |
| FR-21 | Wallet connection | 1.6, 2.9 | ✓ Covered (split) |
| FR-22 | Room terms disclosure | 2.8 | ✓ Covered |
| FR-23 | Schedule and status view | 2.6, 2.10, 3.8, 4.3 | ✓ Covered (split) |

**FRs claimed in epics but absent from the PRD:** none. The epics document uses FR-1…FR-23 exactly as the PRD numbers them, with no invented requirements.

**Deliberate splits, verified as complete rather than partial:**
- **FR-17** — pre-commit maximum exposure in Story 2.3 (the Underwriter cannot decide without it); live exposure updating on each Slash and Backstop draw in Story 3.14 (requires the waterfall to exist).
- **FR-21** — WalletConnect session and balance reading in Story 1.6; exact shortfall against Stake plus first Contribution in Story 2.9 (not computable without Room terms).
- **FR-23** — schedule, positions, dates, and completion state in Story 3.8; Trust Score and its change history in Story 4.3.

Each half of each split carries its own testable acceptance criteria, so no FR is left with a partially specified consequence.

### Missing Requirements

**Critical missing FRs: none.** All 23 PRD functional requirements have a traceable implementation path in at least one story's acceptance criteria.

**Consequence-level gaps (two, both minor).** These are cases where the FR is covered but one specific PRD consequence has no acceptance criterion asserting it:

1. **FR-1 — expired attestation does not remove an in-progress Member.** The PRD states that a Member with an expired attestation "cannot join a *new* Room but is not removed from a Room already in progress." Story 1.3 asserts that an expired attestation is reported rather than deleted and stays readable; Story 2.5 asserts that a join with an expired attestation reverts. Neither asserts the negative — that an in-progress membership survives expiry. *Recommendation: add an acceptance criterion to Story 2.5 or 3.1 stating that attestation expiry mid-Cycle has no effect on an existing membership, contribution obligation, or Payout entitlement.*
2. **FR-20 — the platform takes no share of the fee and no share of the Pot.** Stories 2.8 and 3.1 assert that the fee is charged on Contribution rather than Payout and that the Pot is never reduced. Neither states the platform's zero share explicitly, which is a distinct claim — it is the monetization boundary in PRD §11 and part of the "software, not a financial counterparty" regulatory position in §8. *Recommendation: add an acceptance criterion to Story 3.1 asserting that no platform address receives any portion of a Contribution, fee, or Pot on any code path.*

Both are additive single-criterion fixes to existing stories. Neither changes epic structure, story sequencing, or scope.

### Coverage Statistics

- **Total PRD FRs:** 23
- **FRs covered in epics:** 23
- **Coverage percentage:** 100%
- **FRs covered by more than one story:** 22 of 23 (only FR-22 is carried by a single story, Story 2.8, appropriately — it is one screen)
- **Consequence-level gaps identified:** 2 (FR-1, FR-20) — both additive fixes, neither blocking

## UX Alignment Assessment

### UX Document Status

**Found** — a complete `bmad-ux` spine pair, both `final` and dated 2026-08-11:
- `DESIGN.md` — visual identity, design tokens, component visual specs
- `EXPERIENCE.md` — information architecture, behavior, states, interactions, accessibility, journeys
- `mockups/room-terms-commit.html` — composition reference, subordinate to both spines by their own stated precedence rule

The pair is complete, so no partial-handoff decision was required. `EXPERIENCE.md` maps each of its 18 surfaces to the FRs it realizes, which made UX↔PRD traceability directly checkable rather than inferred.

### UX ↔ PRD Alignment

**Strong.** The UX spine's journeys mirror UJ-1…UJ-4 verbatim in protagonist and framing, its IA table carries an FR column per surface, and its product-specific sections (Money Legibility, Irreversibility & Commitment, Trust Score Legibility) are derived from FR-12, FR-13, FR-16, FR-20, FR-22, and §9 rather than invented.

Findings:

1. **Two surfaces have no originating PRD journey — self-flagged, not a defect.** `EXPERIENCE.md`'s "Closure note" states that Account and Activity exist because a WalletConnect session and an irreversible on-chain history need somewhere to live, not because a stated need produced them. Flagging rather than back-filling an invented journey is the correct handling. Both are covered by Story 1.8.
2. **The UX spine narrows FR-14 rather than realizing it.** FR-14 requires the Member be notified at Grace Window open and at 24 hours remaining. `EXPERIENCE.md § Notification & Reach` records a founder decision of 2026-08-11 to ship in-app notification only, which means the requirement is satisfied only for a Member who opens the app — and the Member this exists for is precisely the one whose salary is late and whose app is closed. The spine names this as the product's weakest link rather than concealing it. *Status: accepted risk with a recorded pre-mainnet escalation (SMS), carried into Story 5.6 as a decision that must be closed in writing.*
3. **FR-22 versus FR-11 on the terms screen** — FR-22 requires the pre-commit screen to show the Member's Payout Position; FR-11 computes positions only at Room start; `EXPERIENCE.md` Flow 1 shows "payout position 6" pre-fill. *Status: resolved during epic creation as a provisional position with the ranking rule stated, recorded as a note at the head of Epic 2 and specified in Story 2.8.*
4. **The indicative peso line is a UX addition beyond the PRD.** The PRD requires an FX disclosure and a recorded acknowledgement (FR-22); it does not contemplate displaying peso conversions. `EXPERIENCE.md § Money Legibility` adds them, decided 2026-08-11, conditioned on an attribution rule (rate source and timestamp stated on any commitment surface). This is a defensible comprehension decision for a cohort that thinks in pesos — but it creates an architectural dependency the PRD never triggered. See UX↔Architecture finding 1.

### UX ↔ Architecture Alignment

**Aligned on the load-bearing decisions**, with three gaps where the UX requires a capability the architecture does not describe.

Confirmed aligned:
- **Wallet signing model.** `EXPERIENCE.md`'s signing-handoff flow and AD-15's single-`require_auth`-on-source rule are consistent; the UX never requires a Member to sign a detached authorization entry.
- **Chain as authority.** Pull-to-refresh, "as of" timestamps on offline state, and reconstructing pending transaction state from chain rather than memory all follow AD-1 rather than fighting it.
- **Read surfaces.** The schedule, members list, round detail, and exposure views are served by the indexer/API read models (AD-1, AD-7), and none of them is on a path that decides money movement.
- **Accessibility and responsive requirements** are client-layer concerns with no architectural dependency.

Gaps:

1. **No FX rate source exists in the architecture.** `UX-DR10` requires that wherever a peso figure appears on a commitment surface, the rate source and its timestamp are stated. The architecture spine names no price feed, oracle, or rate service in its container view, source tree, or stack table, and no AD governs where a rate comes from or how stale it may be. *Impact: moderate. Story 2.8 and Story 3.9 have acceptance criteria that cannot be satisfied without deciding this. Recommendation: either add a rate source to the architecture (an off-chain read-only adapter is sufficient under AD-7 since no rate ever enters a contract decision), or drop the indicative peso line and keep the dollar figure alone — which the PRD would still satisfy.*
2. **Member handle is undefined.** FR-9 and the UX both identify Members by handle, and `EXPERIENCE.md` is explicit that no avatar, name, or photo may appear. But AD-10 permits only wallet addresses, an opaque provider reference, a tier, and integers on-chain, and nothing in the architecture says what a handle is or where it lives — derived from the address, chosen by the Member and stored off-chain, or something else. *Impact: moderate. Stories 3.10, 2.4, and 1.8 all display handles. Recommendation: specify handle derivation before Story 1.8. A deterministic derivation from the wallet address is the cheapest option consistent with AD-10 and NFR-7; a Member-chosen handle needs an off-chain store and a uniqueness rule, which is a larger decision.*
3. **No frontend framework or PWA tooling is chosen.** The architecture's stack table names `soroban-sdk`, Rust, `stellar-cli`, `@stellar/stellar-sdk`, Stellar Wallets Kit, TypeScript/Node, and PostgreSQL — but nothing for the client. `EXPERIENCE.md` explicitly notes that no UI system is named in the PRD or architecture and that both spines therefore specify from first principles. The architecture's Deferred section does not list this. *Impact: low-to-moderate. Story 1.1 and Story 1.2 must make the choice, and it is not currently anyone's stated decision. Recommendation: record it as an explicit decision at scaffold time rather than letting it be made implicitly by whoever writes Story 1.1.*

### Warnings

- **A stale PRD statement about wallet integration.** FR-21 states that "Soroban contract invocation requires `signAuthEntry`, and that path against Freighter Mobile must go through WalletConnect directly." AD-15 supersedes this: it records that `signAuthEntry` is *not* exposed by Stellar Wallets Kit's WalletConnect module, and therefore designs entry points so that detached auth entries are never needed. The architecture's position is the correct one and the epics follow it, but the PRD sentence now reads as a requirement for something the design deliberately avoids. *Recommendation: amend FR-21's rationale in the PRD so a reader does not implement toward `signAuthEntry`.*
- **The UX spine's own assumptions remain unratified.** `DESIGN.md` states that the entire aesthetic direction is a first proposal derived from a temperament and a cohort description, with no brand assets or prior visual work supplied. This is honestly flagged, but it means the token values in Story 1.2 are provisional, and a later brand decision would ripple through every UI story.
- **No design system inheritance point exists.** `EXPERIENCE.md` states that if a UI system is chosen before build, its section is where that inheritance gets recorded and the component tables collapse to deltas. That has not happened, so the component specs are absolute rather than deltas — which is more work in Story 1.2 but removes a dependency.

## Epic Quality Review

Assessed against the `bmad-create-epics-and-stories` standards. **Reviewer note:** `epics.md` was authored in this same session, so this section is partly self-review. Findings against it are stated as directly as findings against the upstream documents, but an independent adversarial pass would still be worth running.

### Epic Structure — User Value

| Epic | Title user-centric? | Goal describes user outcome? | Valuable alone? | Verdict |
|---|---|---|---|---|
| 1. A verified member with a working wallet | Yes | Yes — a person can connect, verify, and see their score | Yes | ✓ Pass |
| 2. A room you can read completely before you commit | Yes | Yes — terms are legible and commitment is a single act | Dev-complete, see 🟡-3 | ⚠ Qualified |
| 3. A cycle that pays out on schedule | Yes | Yes — the product's central promise | Yes | ✓ Pass |
| 4. Trust that compounds | Yes | Yes — the portable asset the product claims to create | Yes | ✓ Pass |
| 5. Pilot readiness — testnet to mainnet | No | No — describes a gate, not a user capability | As a gate only | ❌ See 🟠-1 |

### Epic Independence

Epic 1 stands alone. Epic 2 uses only Epic 1's Registry. Epic 3 uses Epic 1 and 2 outputs. Epic 4 depends on Epic 1's Registry plus events Epic 3 emits, and its computation story can be built against fixture events. Epic 5 gates mainnet rather than development. **No epic requires a later epic to function.** No circular dependencies.

### Findings by Severity

#### 🔴 Critical Violations

**None.** No forward dependencies that break a story's completability, no epic-sized stories, and no epic that cannot be built in its stated order.

#### 🟠 Major Issues

**🟠-1. Epic 5 is a technical/milestone epic, not a user-value epic.** "Pilot readiness — testnet to mainnet" covers no FR, and its stories are gates (audit-readiness review, licence headers, deployment, cost measurement, exposure table). By the standard's own red-flag list this is the "Infrastructure Setup" pattern. *Counter-argument, recorded at design time and approved by the user:* NFR-1 is an absolute precondition with no exception path, and open-sourcing is part of the trust proposition rather than a release chore — leaving them out of the tracked breakdown risks them being done informally or not at all. *Recommendation: keep it if you want these tracked as work; move Stories 5.1–5.7 to a milestone or release checklist if you want the epic list to remain strictly user-value. Either is defensible; what is not defensible is leaving NFR-1 untracked.*

**🟠-2. No CI/CD or test-harness story exists anywhere in the breakdown.** The standard's greenfield checklist expects development environment configuration and CI/CD early. Story 1.1 establishes the source tree, toolchain, and binding generation, and mentions a lint/CI check to keep `bindings/` un-hand-edited — but no story stands up a pipeline, and no story establishes the contract test harness (Soroban test utilities, unit test conventions) or an end-to-end test approach. This matters more than usual here: NFR-1 requires an audit-readiness review, Story 5.3 asserts invariant properties across the whole contract surface, and SM-2 requires an induced Default resolved through the waterfall — none of which has a stated testing vehicle. *Recommendation: add a story to Epic 1 covering the contract test harness and a CI pipeline that builds contracts, regenerates bindings, and runs tests on every change. Story 5.3's invariant assertions should be expressed as tests in that harness rather than as a manual review.*

**🟠-3. Story 2.2 contains a forward-referencing acceptance criterion.** Its AC "Given a Room that has closed with Backstop undrawn / When close completes / Then the undrawn remainder returns to the Underwriter" describes behavior implemented by Story 3.6 in a later epic. Story 3.6 already carries an equivalent criterion. As written, Story 2.2 cannot be fully verified on completion. *Recommendation: remove that AC from Story 2.2, or restate it as a state invariant ("Backstop is held by the Room and is withdrawable only via the close path") that is testable within Epic 2.*

#### 🟡 Minor Concerns

**🟡-1. Three stories have thin user value and are effectively infrastructure.** Story 1.1 (scaffold, written "As a developer"), Story 1.2 (tokens and PWA shell), and Story 3.7 (read models and API). Each is justified — 1.1 because no starter template exists and the workflow anticipates it as Epic 1 Story 1; 1.2 because every UI story depends on the token layer; 3.7 because whole-cycle views need read models — but all three would be flagged by a strict reading of the standard. *No change recommended; noted for transparency.*

**🟡-2. Story 1.2 is the largest sizing risk in the breakdown.** It bundles the full token system, the PWA shell, responsive behavior, three component primitives, and the accessibility floor into one story. It may exceed a single dev session. *Recommendation: if it runs long, split the accessibility floor and responsive shell from the token layer — the split is clean and neither half forward-depends on the other.*

**🟡-3. Epic 2 is dev-independent but not independently deployable with real value.** `epics.md` marks it "Standalone: yes," which is true in the sense the standard means — it delivers complete functionality for its domain and needs no later epic to build. But a Room that has formed and frozen with Member funds locked has no way to run Rounds or return those funds except open-window cancellation. *Recommendation: no structural change; add a line to Epic 2's notes stating it must not be deployed to mainnet ahead of Epic 3, so "standalone" is never read as "shippable."*

**🟡-4. Story 1.7 displays a stake multiple before Story 2.5 implements stake sizing.** The FR-13 formula is a pure function and can be computed client-side, so this is not a blocking forward dependency — but the formula would then exist in two places. *Recommendation: define the stake formula once in shared client code in Story 1.7 and have Story 2.5's UI reuse it, or drop the multiple from the verification-status screen and show it first on the terms screen.*

**🟡-5. Story 2.6's Fairness Floor criterion is unexercisable in V1.** The rule concerns a Member holding the final Payout Position in two preceding Cycles *within the same Room*, and V1 runs one Room for one Cycle. The AC is correct per FR-11 and should be implemented and unit-tested, but it cannot be validated by the pilot. The PRD acknowledges the wider version of this in §13 item 8. *No change recommended; flagged so it is tested rather than assumed exercised.*

**🟡-6. Some acceptance criteria are implementation-phrased rather than behavior-phrased.** Examples: "When the app's styling layer is implemented," "When a lint or review check runs over the styling layer," "When the contract source is searched for randomness." These are verifiable but read as review activities rather than system behavior. *Acceptable for constraint-type criteria; noted as a consistency deviation.*

### Best Practices Compliance Checklist

| Check | Result |
|---|---|
| Epic delivers user value | 4 of 5 pass; Epic 5 is a gate epic (🟠-1) |
| Epic can function independently | Pass — no epic requires a later one |
| Stories appropriately sized | Pass, with Story 1.2 flagged (🟡-2) |
| No forward dependencies | One violation found (🟠-3, Story 2.2) |
| Entities/state created when needed | Pass — Registry state in 1.3–1.4, Room state in 2.1, escrow in 3.1, read models in 3.7 |
| Clear acceptance criteria | Pass — all 44 stories use Given/When/Then; 🟡-6 notes phrasing drift |
| Traceability to FRs maintained | Pass — 23/23 FRs traceable to acceptance criteria |
| Starter template handled correctly | Pass — architecture specifies none, Story 1.1 scaffolds from toolchain |
| Greenfield setup expectations | **Fail** — no CI/CD or test harness story (🟠-2) |

## Summary and Recommendations

### Overall Readiness Status

**NEEDS WORK** — but narrowly, and not structurally.

Requirements traceability is complete (23/23 FRs, 100%), epic structure holds up, no epic depends on a later one, and there are zero critical violations. What holds the status back is a small set of concrete gaps: one missing capability class (test harness and CI), three architecture decisions the UX silently assumes, and one forward-referencing acceptance criterion. Every fix is additive. None requires re-cutting epics, re-sequencing stories, or revisiting the PRD's substance.

**Epic 1 Story 1.1 can begin immediately.** The blocking items land in Stories 1.2, 1.7, 1.8, 2.2, 2.8, and 3.9, so they need resolving within days rather than before the first line of code.

### Critical Issues Requiring Immediate Action

No 🔴 critical issues. The three 🟠 major issues, in the order they will bite:

1. **No test harness or CI pipeline story exists (🟠-2).** This is the most consequential finding. NFR-1 makes an audit-readiness review an absolute precondition for mainnet; Story 5.3 asserts invariant properties across the entire contract surface; SM-2 requires an induced Default resolved through the full waterfall. None of these has a stated testing vehicle. For a product whose dominant quality attribute is funds safety, the absence of a testing story is the single largest gap in the breakdown.
2. **Three architecture decisions the UX assumes but the architecture never made:** no FX rate source (blocks acceptance criteria in Stories 2.8 and 3.9), Member handle undefined (blocks Stories 1.8, 2.4, 3.10), and no frontend framework or PWA tooling chosen (blocks Stories 1.1 and 1.2).
3. **Story 2.2 has a forward-referencing acceptance criterion (🟠-3)** describing Room-close behavior that Story 3.6 implements, making Story 2.2 unverifiable at completion.

### Recommended Next Steps

1. **Add a test-harness and CI story to Epic 1** — contract test conventions using Soroban test utilities, a pipeline that builds contracts, regenerates bindings, and runs tests on every change. Re-express Story 5.3's invariant assertions as tests in that harness rather than as a manual review pass.
2. **Decide the Member handle** before Story 1.8. Deterministic derivation from the wallet address is the cheapest option consistent with AD-10 and NFR-7; a Member-chosen handle requires an off-chain store and a uniqueness rule.
3. **Decide the FX rate question** before Story 2.8. Either add a read-only rate adapter to the architecture (permitted under AD-7, since no rate ever enters a contract decision) or drop the indicative peso line — the PRD is satisfied either way, but `UX-DR10` as written is not implementable without a source.
4. **Record the frontend framework and PWA tooling choice** in the architecture stack table, so Story 1.1 implements a decision rather than making one.
5. **Fix Story 2.2's forward-referencing AC** — remove it, or restate it as a state invariant testable within Epic 2.
6. **Add the two missing consequence-level criteria:** FR-1's "attestation expiry does not affect an in-progress membership" (Story 2.5 or 3.1) and FR-20's "no platform address receives any share of Contribution, fee, or Pot" (Story 3.1).
7. **Amend FR-21's rationale in the PRD** so its `signAuthEntry` statement no longer contradicts AD-15, which supersedes it.
8. **Decide Epic 5's fate** — keep it as a tracked gate epic, or move its stories to a release checklist. Either is defensible; leaving NFR-1 untracked is not.

### Final Note

This assessment identified **20 issues across 4 categories** — 0 critical, 3 major, 6 minor, and 11 documentation or alignment observations (3 PRD completeness gaps, 2 FR consequence gaps, 3 UX↔Architecture gaps, 3 UX warnings). Address the major issues before Epic 1 progresses past its scaffold story. These findings can be used to improve the artifacts, or you may choose to proceed as-is.

The planning artifacts are, on the whole, in unusually good shape: a PRD whose every requirement carries testable consequences, an architecture whose invariants are stated as prohibitions rather than intentions, and a UX spine that names its own weakest link instead of hiding it. The gaps found here are the ordinary seams between four documents written at different altitudes — not defects in any one of them.

---

## Remediation Log — applied 2026-08-12

Five epic-level fixes were applied to `epics.md` immediately following this assessment. The report body above is preserved as the point-in-time finding; this section records what has since been closed.

**Story renumbering:** inserting the new test-harness story as Story 1.2 shifted the rest of Epic 1. Old 1.2→1.3, 1.3→1.4, 1.4→1.5, 1.5→1.6, 1.6→1.7, 1.7→1.8, 1.8→1.9. Story references in the report body above use the **pre-renumbering** numbers. Epic 1 now has 9 stories; the breakdown totals 45.

| Finding | Action taken | Status |
|---|---|---|
| 🟠-2 CI/test harness missing | Added **Story 1.2 — Contract test harness and CI pipeline** (7 ACs): Soroban unit-test conventions asserting reverts by error code, full-Cycle integration test including an induced Default through the waterfall, ledger-timestamp time control, CI that builds contracts and fails on stale bindings, deterministic test runs, and the requirement that Story 5.3's invariants be expressed as tests. Story 5.3 gained a criterion binding it to this harness. | ✅ Closed (epics side) |
| 🟠-3 Story 2.2 forward-referencing AC | Replaced the Room-close AC with a state invariant testable within Epic 2: the only paths moving Backstop are the waterfall draw and the close return, with no other exit. Story 3.6 retains the close-return behavior. | ✅ Closed |
| FR-1 consequence gap | Added an AC to Story 3.1: attestation expiry mid-Cycle leaves Contribution obligations and Payout entitlement intact — expiry blocks joining a *new* Room only. | ✅ Closed |
| FR-20 consequence gap | Added an AC to Story 3.1: no platform or operator address receives any portion of a Contribution, fee, or Pot on any code path. | ✅ Closed |
| 🟡-3 Epic 2 "standalone" ambiguity | Epic 2's notes now distinguish development independence from deployability, and state that Epic 2 must not reach mainnet ahead of Epic 3. | ✅ Closed |

**Still open after remediation:**

- **The three architecture decisions** — FX rate source, Member handle derivation, frontend/PWA stack. These belong in `ARCHITECTURE-SPINE.md`, not in `epics.md`, and were deliberately not patched here. Until they land, Stories 1.3, 1.8, 1.9, 2.4, 2.8, 3.9, and 3.10 carry acceptance criteria that depend on undecided inputs.
- **PRD FR-21's stale `signAuthEntry` rationale** — belongs in the PRD.
- **🟠-1 Epic 5's disposition** — a judgment call for the founder, unchanged.
- **🟡-1, 🟡-2, 🟡-4, 🟡-5, 🟡-6** — accepted as noted; no action taken.

**Revised status after remediation: NEEDS WORK** — unchanged, because the three architecture decisions remain open and they are what gate story implementation. The epics-side findings are now closed.

### Second remediation pass — same day, after the architecture, UX, and PRD updates

All three architecture decisions were subsequently made and propagated. Status is now **READY**.

| Item | Resolution |
|---|---|
| FX rate source | **AD-16** — read-only rate adapter in `services/rates/`, presentational only, never an input to a contract decision. UX set the freshness window at one hour; the API omits an out-of-window rate and the client renders no peso line, with nothing in its place. |
| Member handle | **AD-17** — pure deterministic function of the Stellar address, living in exactly one shared module. UX chose a short alphanumeric code, grouped, excluding `0`/`O` and `1`/`I`, in a new `handle` typography role. |
| Frontend stack | React + Vite + `vite-plugin-pwa`, recorded in the architecture stack table. No component system adopted; the resulting hand-built accessibility burden is recorded in `EXPERIENCE.md § Foundation` as a named build risk. |
| PRD FR-21 | Amended with a visible `[AMENDED 2026-08-12]` bullet naming AD-15 as the superseding decision. The WalletConnect requirement stands; only its justification moved. |

**Two defects the follow-on reviewer gates caught, neither visible to this assessment:**

- **Architecture (adversarial lens):** AD-16 and AD-17 as first drafted each fixed a *property* without fixing a *source* — two conforming implementations could still diverge (a word-pair handle in the client versus a base32 handle in the indexer; a client fetching its own rate; staleness judged in two layers). All three tightened.
- **UX (rubric walker):** `state-grace` `#A5701A` measured 3.86:1 on its own wash and 4.26:1 on raised, failing WCAG AA as body text everywhere, while `EXPERIENCE.md` held the grace banner to AAA. Darkened to `#6B4710` (7.51:1 / 8.29:1). A measured-contrast table was added to `DESIGN.md § Colors`, since the absence of numbers was the root cause. Three missing cold states were also added, including Trust score with no history — the state every Member is in at pilot start, on the surface SM-5 measures.

**Story sharpening pass.** The seven stories whose acceptance criteria depended on the then-undecided inputs were updated to cite the actual decisions: Stories 1.1, 1.3, 1.9, 2.4, 2.8, 3.7, 3.9, 3.10, and 4.3. Two new requirements were added to the inventory — **UX-DR26** (handle derivation and legibility) and **UX-DR27** (cold and empty states) — and UX-DR1 and UX-DR10 were corrected. The breakdown stands at 45 stories with no duplicate numbering.

**Remaining open items are all decisions rather than defects:** Epic 5's disposition, the absence of a dedicated FR-16 removal journey, the mock's unnamed rate source pending provider selection, and the five accepted 🟡 minors.

---

**Assessment date:** 2026-08-12
**Assessor:** Product Manager (implementation readiness workflow)
**Caveat:** `epics.md` was authored in the same session as this assessment, so the Epic Quality Review section is partly self-review. An independent adversarial review of `epics.md` is recommended before sprint planning.
