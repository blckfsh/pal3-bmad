---
stepsCompleted:
  - "step-01-validate-prerequisites"
  - "step-02-design-epics"
  - "step-03-create-stories"
inputDocuments:
  - "_bmad-output/planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md"
  - "_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/DESIGN.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/EXPERIENCE.md"
  - "_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/mockups/room-terms-commit.html"
---

# paluwagan3 - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for paluwagan3 (Pal3), decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

Numbering is preserved verbatim from the PRD (FR-1…FR-23) so downstream references stay stable.

**Identity and Verification**

- **FR-1: Member identity verification** — A prospective Member can complete identity verification through the KYC Provider before joining any Room. A wallet with no valid KYC Attestation cannot join; Pal3 stores only attestation status, tier, issue date, and provider reference; an expired attestation blocks new Rooms but does not remove a Member from a Room in progress.
- **FR-2: Verification tier sets starting Trust Score** — Basic verification yields 300, full verification yields 400, and no Member exceeds 400 until at least one Cycle is complete.
- **FR-3: One human, one identity** — Provider duplicate-detection is enforced at attestation issue; a second attestation for the same individual is rejected and a join from a duplicate-flagged wallet reverts.

**Room Lifecycle**

- **FR-4: Room creation** — An Underwriter creates a Room specifying Contribution amount, Cadence, and Member count, bounded by subscription tier capacity, requiring posted Backstop per FR-19, weekly Cadence only in V1.
- **FR-5: Member admission** — An Underwriter reviews and admits or rejects applicants on Trust Score, Cycles completed, Default count, and account age only — never identity. Rejected applicants' locked funds return in full.
- **FR-6: Room start** — A Room starts automatically and only when it reaches its configured Member count. Payout Positions freeze at start; parameters become immutable; failure to fill within the open window (14 days) cancels the Room and returns every Stake and Contribution in full.
- **FR-7: Room completion** — A Room closes after its final Round, returning every completing Member's full Stake and applying cycle-completion Trust Score credit.

**Contributions and Escrow**

- **FR-8: Scheduled contribution** — A Member pays their Contribution for the current open Round and the contract escrows it. Escrowed funds leave only via Payout, Slash, or cancellation refund. No double-payment within a Round.
- **FR-9: Contribution status visibility** — Any Member sees every Member's current-Round status — paid, pending, in Grace Window, or defaulted — identified by handle, never legal identity.

**Trust Engine**

- **FR-10: Trust Score computation** — A 0–1000 score per Member updated on defined events: +10 on-time, +2 cured in Grace, −150 Default, +50 Cycle completed with zero Defaults, −400 abandonment via Stake exhaustion. Clamped to [0, 1000], deterministic and reproducible from event history.
- **FR-11: Payout ordering with Fairness Floor** — Positions assigned by descending Trust Score at Room start; ties broken deterministically by earlier Room-join timestamp with no randomness; a Member cannot hold the final position a third consecutive Cycle in the same Room; positions visible before start and frozen thereafter.
- **FR-12: Pot payout** — The contract releases the full Pot (Member count × Contribution) to the current Round's position holder with no platform deduction, on the scheduled boundary regardless of Default events, exactly once per Member per Cycle.
- **FR-13: Stake sizing** — Stake = Contribution × (2 − TrustScore/1000), computed at join, displayed with the score that produced it before commitment, and not recomputed mid-Cycle.

**Default Handling**

- **FR-14: Grace Window** — A missed deadline opens a fixed 48-hour window during which payment is accepted as cured rather than Defaulted. The Round does not advance and no Payout occurs while open; the Member is notified at open and at 24 hours remaining; other Members see "awaiting contribution" with time remaining.
- **FR-15: Slash and Backstop waterfall** — On Default the contract slashes exactly the missed Contribution from the defaulter's Stake, draws the remainder from the Backstop, pays the recipient the full Pot in the same Round with no delay or reduction in every branch, applies the Default penalty, and advances the Round.
- **FR-16: Stake exhaustion and removal** — A Member whose Stake reaches zero is removed at that moment; the Backstop carries their remaining Rounds; an unreceived Payout is forfeited while an already-received one is retained with the Backstop absorbing the loss; the abandonment penalty applies.
- **FR-17: Underwriter exposure visibility** — Maximum exposure is computable and displayed before the Underwriter commits; live exposure updates on every Slash and Backstop draw.

**Underwriter Subscription and Capacity**

- **FR-18: Tiered capacity** — Subscription tier caps concurrent Rooms and Members per Room; requests exceeding either revert. V1 operates a single tier: one Room, maximum 10 Members.
- **FR-19: Capital adequacy gate** — Required Backstop is a published function of Member count and Contribution (minimum 2 × Contribution × Member count). Member admission cannot open until it is posted and locked, and it cannot be withdrawn while any Room it collateralizes is in progress.
- **FR-20: Per-contribution fee** — The Underwriter earns a fee per Contribution, displayed as both percentage and absolute per-Round amount before a Member commits, charged on Contribution never on Payout, with no platform share of fee or Pot.

**Member Application**

- **FR-21: Wallet connection** — A Member connects a Stellar wallet from mobile via WalletConnect (not a browser-extension API) and funds it with the Room's stablecoin; at least one WalletConnect-compatible wallet works end to end; insufficient balance surfaces the exact shortfall before a join is attempted.
- **FR-22: Room terms disclosure** — One screen shows Contribution, Cadence, Member count, Payout Position and date, Stake, Underwriter fee, and the default waterfall in plain language; states explicitly that funds are USD-pegged and not pesos and that peso value may move; requires a recorded FX acknowledgement before joining.
- **FR-23: Schedule and status view** — A Member sees all Rounds, all Payout Positions and dates, which Rounds are complete, and their own Trust Score with its change history.

### NonFunctional Requirements

- **NFR-1: Funds safety is the dominant quality attribute.** Contracts holding escrow, Stake, and Backstop must pass an audit-readiness review before any mainnet Room opens with real value. No mainnet pilot begins without it.
- **NFR-2: Determinism.** Payout ordering, Slash amounts, and Trust Score changes must be reproducible from event history by any observer. No randomness anywhere in the system.
- **NFR-3: No privileged override.** No admin key, upgrade path, or operator action can move escrowed funds, alter a Payout Position mid-Cycle, or reverse a Slash. This outranks operational convenience.
- **NFR-4: Availability at Round boundaries.** Contribution and Payout must succeed at scheduled boundaries. Off-chain trust-service unavailability must never block an on-chain Payout.
- **NFR-5: Cost per Round.** Transaction cost must stay negligible relative to a ₱1,000-equivalent Contribution — a product constraint, not only an engineering one.
- **NFR-6: KYC boundary.** Identity document handling is outside Pal3's technical boundary by design; the KYC Provider is the sole personal information controller for document data.
- **NFR-7: No personal data on-chain.** Wallet addresses, Trust Scores, and Room state are pseudonymous; the link to legal identity exists only in the KYC Provider's records.
- **NFR-8: Irreversibility disclosure.** Trust Score history is append-only and a Default is permanent; Members must be told before joining that this record cannot be erased.
- **NFR-9: Open source.** All Soroban contract source is published under Apache 2.0 from first deployment.
- **NFR-10: Surface constraint.** Mobile-first PWA rather than native for V1, to avoid app-store review inside the eight-week window.
- **NFR-11: Portability of the Trust Engine.** The Trust Engine is built as chain-agnostic off-chain service code so the mechanism survives a later migration toward a peso rail.

### Additional Requirements

Derived from the Architecture Spine (AD-1…AD-15), Consistency Conventions, Stack, and Structural Seed.

**Starter template:** the Architecture specifies **no greenfield starter template**. It specifies a toolchain (`stellar-cli` for `contract build` and `contract bindings`, Rust 1.84+ targeting `wasm32v1-none`, soroban-sdk 25.0.0, TypeScript/Node for services and client) and a fixed source tree. Epic 1 Story 1 is therefore a **scaffold-from-toolchain** story, not a template-clone story: create the `contracts/` (registry, factory, room, shared), `services/` (trust, kyc, indexer, api), `app/`, and `bindings/` layout, pin `stellar-cli`, `@stellar/stellar-sdk`, and Stellar Wallets Kit versions at install and record them.

- **AR-1 (AD-1):** The ledger is the only system of record. Any value determining money movement or Member obligation is read from chain state at decision time; off-chain stores are rebuildable caches and never an input to a contract decision.
- **AR-2 (AD-2):** Exactly three contract kinds — Registry (one, network-wide: attestations and Trust Scores), Factory (one: deploys Rooms, enforces capacity and Backstop gates), Room (one instance per Room: escrow, Stakes, schedule, Payouts). A Room reads Registry; Registry never reads a Room; no Room references another Room.
- **AR-3 (AD-3):** All value-bearing state uses `persistent` storage with TTL extended on every state-mutating invocation. `temporary` storage is forbidden for any data whose loss would affect funds or obligations; `instance` holds only per-invocation configuration.
- **AR-4 (AD-4):** Entry expiry is never control flow. All time-based transitions compare ledger timestamp against a deadline stored in Room state and are effected by explicit invocation. No contract logic branches on whether an entry has expired.
- **AR-5 (AD-5):** The Room snapshots a Member's Trust Score from Registry at join; the join transaction carries the score the Member was shown and reverts on mismatch. The Room computes Payout Positions itself at start from its snapshots. No off-chain component ever submits an ordering.
- **AR-6 (AD-6):** No address holds authority to transfer escrow, alter a Payout Position after start, or reverse a Slash. Registry admin authority is limited to attestations and Trust Score writes and can never touch Room funds. Any upgrade mechanism must be incapable of altering an in-flight Room.
- **AR-7 (AD-7):** Trust and KYC services write to Registry only between Rooms, never during an active Cycle. Indexer and API are read-only. A total off-chain outage must leave every in-flight Room fully operable by direct contract invocation.
- **AR-8 (AD-8):** TypeScript contract clients are generated via `stellar contract bindings` as part of the contract build. Hand-written encoding of contract types in client code is prohibited; `bindings/` is never hand-edited.
- **AR-9 (AD-9):** Slash, Backstop draw, and Payout for a Round execute in a single invocation that fully succeeds or fully reverts. A Round advances only after the recipient holds the full Pot.
- **AR-10 (AD-10):** Contracts store only wallet addresses, an opaque KYC provider reference, an attestation tier, and integers. No name, document, employer, bank detail, or hash derived from a raw identity document is written on-chain.
- **AR-11 (AD-11):** No randomness anywhere. Ordering, Stake sizing, Slash amounts, and Trust Score changes are pure functions of committed state and ledger timestamp.
- **AR-12 (AD-12):** All Soroban contract source published under Apache 2.0 from first deployment.
- **AR-13 (AD-13):** `advance_round` is callable by any address once the ledger timestamp passes the Round deadline; it is idempotent, reverts if called early, and its effect is identical regardless of caller. An optional keeper is a convenience only — Room liveness must never depend on it.
- **AR-14 (AD-14):** The Factory transfers the required Backstop into the Room contract at creation, before admission opens. All Backstop draws are local to the Room; undrawn Backstop returns to the Underwriter at close; the Factory never custodies value beyond the creation transaction.
- **AR-15 (AD-15):** Entry points are designed so the authorized address is the transaction source — a single `require_auth` on the invoker, satisfied by signing the transaction. No entry point may require a detached auth entry (`signAuthEntry` is unavailable through Stellar Wallets Kit's WalletConnect module). Multi-party authorization within one invocation is prohibited in V1.
- **AR-16 (Naming):** Contract types and functions use PRD Glossary terms verbatim — `Room`, `Member`, `Underwriter`, `Contribution`, `Pot`, `Stake`, `Backstop`, `TrustScore`, `PayoutPosition`. A synonym in code is a defect.
- **AR-17 (Money):** All amounts are integer stroops of the Room's stablecoin asset. No floating point in contracts or services; amounts are never formatted for display below the client layer.
- **AR-18 (Time):** Ledger timestamp in seconds is the sole time source in contracts. Cadence and Grace Window are stored as second offsets. Contracts never receive a client-supplied time.
- **AR-19 (Identifiers):** `Member` is keyed by Stellar address; `Room` by its deployed contract address; attestations carry an opaque provider reference string, never a derived identity hash.
- **AR-20 (Errors):** One `#[contracterror]` enum per contract with stable discriminants, never renumbered once deployed. Clients map codes to copy and never parse messages.
- **AR-21 (Events):** Every state transition that moves value or changes a Trust Score emits an event. The indexer is built only from events, never from state polling, so read models stay replayable.
- **AR-22 (Trust Score service):** Computed off-chain as a pure, versioned function of chain event history; the version is recorded with each commit to Registry.
- **AR-23 (Config):** Contract addresses and network passphrase come from environment at build/deploy time and are never hard-coded in client or service source.
- **AR-24 (Read models):** PostgreSQL holds read models only — rebuildable, never authoritative.
- **AR-25 (Deploy path):** Testnet first, then Mainnet, with the audit-readiness review (NFR-1) gating any mainnet Room holding real value.

### UX Design Requirements

Extracted from the `DESIGN.md` / `EXPERIENCE.md` spine pair. `DESIGN.md` owns visual identity; `EXPERIENCE.md` owns IA, behavior, states, and accessibility.

**Design tokens and foundations**

- **UX-DR1:** Implement the full design token set from `DESIGN.md` frontmatter as the single source of styling truth: 16 color tokens (3 surfaces, 4 ink weights, 2 accents, 2 borders, 3 money states, 2 washes), 8 typography roles (`display`, `title`, `body`, `body-strong`, `meta`, `amount`, `amount-inline`, `handle`), 5 radii, and the spacing scale including the named `margin-mobile` (16px), `gutter` (12px), and `commit-gap` (32px). The measured-contrast table in `DESIGN.md § Colors` is part of the contract: a token substitution that breaks a stated ratio is a defect, not a style change.
- **UX-DR2:** Enforce the color-meaning rules in code: accent (`#124F4B`) only for commitment, trust-score fill, and current-round marker; `state-paid`/`state-grace`/`state-default` reserved absolutely to money states; the two wash colors used only behind the grace banner and a defaulted row; no other background fills; no color at all on trust-score history rows.
- **UX-DR3:** Set every monetary figure in `tabular-nums` via the `amount`/`amount-inline` roles. Type must scale with browser font-size settings; no layout-level `px` locking.
- **UX-DR4:** No shadows, gradients, illustration, mascots, or floating cards anywhere. Raised surfaces are distinguished by tone plus a hairline border; a scrim on sheets and modals is the only depth cue in the system.
- **UX-DR5:** Enforce `commit-gap` (32px) as the minimum vertical distance between the last disclosure element and any control that commits funds, on every funding surface.

**Component library**

- **UX-DR6:** Build the ten spine components to their combined visual and behavioral specs: **Button (primary)** (52px min height, one per surface, full-bleed on commit screens, act + amount label), **Button (secondary)** (hairline outline, never competing), **Amount display** (dollar primary + optional `≈` peso `meta` line, never animates or counts up), **Status pill** (outline only, never filled, four states — paid, pending, in grace, defaulted — with text carrying meaning), **Terms row** (label/value with hairline divider, never truncates, tap opens a plain-language sheet), **Schedule row** (round, date, recipient handle, amount; 3px accent marker on current round; own round in `body-strong`; never reorders), **Member status row** (handle + pill only, no avatar/name/photo), **Trust score panel** (score + fill + consequence, never fewer than all three), **Notice banner** (grace and default-resolved variants only, persistent, not dismissible), **Disclosure block** (sunken, full ink, never collapsed behind "read more").
- **UX-DR7:** No informational or success banner variants, no badges, no streaks, no levels, no grades — the trust score is a financial record and must never render as gamification or as a decoration next to a handle.

**Information architecture and surfaces**

- **UX-DR8:** Build the 20 specified surfaces: Welcome, Connect wallet, Verify identity, Verification status, Room terms, Funding check, Commit, Room filling, Room home, Contribute, Members, Round detail, Trust score, Cycle summary, Account, Activity, Create room, Post backstop, Applicants, Underwriter room — each mapped to the FRs listed in `EXPERIENCE.md § Information Architecture`.
- **UX-DR9:** Implement role gating in one application: after wallet connection the app resolves whether the address underwrites a Room and reveals the **Underwrite** tab. Member tabs are Room / Trust / Account; underwriters gain a fourth. No drawer; sheets stack one level deep only.

**Money legibility (highest-harm surface)**

- **UX-DR10:** The dollar figure is the number of record everywhere. An indicative peso line may appear beneath it in `meta`, always prefixed `≈`, never inside a button label and never on the FX acknowledgement itself. Attribution — rate source **and** observation timestamp — is stated once per commitment surface, beneath the primary amount, governing every peso figure on that surface; a surface with no attribution anywhere shows no peso figures at all. The freshness window is one hour: the API omits an out-of-window rate (AD-16), the client then renders no peso line, and **nothing takes its place** — no placeholder, no "rate unavailable", no dash.
- **UX-DR26:** Display every Member by a handle derived from their Stellar address by one shared module consumed by client and indexer alike (AD-17) — never chosen or changed by the Member, never accompanied by copy implying it is a name. It renders in the `handle` typography role with tabular figures, is grouped rather than run together, excludes `0`/`O` and `1`/`I`, and is announced by screen readers character by character rather than as an attempted word.
- **UX-DR27:** Specify the cold and empty states: **Trust score with no history** — the state every Member is in at pilot start and the surface SM-5 measures — states that nothing has moved the score yet and names the first event that will, never an illustration or encouragement; **Activity with no actions** states that actions appear once taken, with no placeholder rows; **Applicants with no requests** states that admission is open and how many Members the Room still needs, rather than "no results".
- **UX-DR11:** Label the four amounts with their own noun in every context — *contribution*, *stake*, *pot*, *fee*. The pot always displays its full undeducted value; no "net payout" figure may exist anywhere. Stake always displays as multiple and amount together ("1.6× one contribution — $28.80").

**Irreversibility and commitment**

- **UX-DR12:** Implement the five-part commit pattern on all three irreversible actions (join, contribute, admit applicant): what you give up as an amount → what you get and when → what happens if you can't pay later → `commit-gap` → one primary action labeled with act and amount ("Lock $46.80 and join", never "Confirm"/"Continue").
- **UX-DR13:** No confirmation dialogs on commit screens, no undo affordance, no toast offering one. Disclose FR-16 removal-on-stake-exhaustion at join, not at removal.
- **UX-DR14:** Keep the commit control disabled until the FX acknowledgement is checked, and state the blocker in text beneath a disabled primary action — never in a tooltip.

**Trust score legibility (measured by SM-5)**

- **UX-DR15:** Present score, consequence, and history in that order, always together. Every event is a plain sentence with its delta ("Paid round 3 on time · +10 · 24 September") — no icons, no categories, no aggregation. State the next threshold in member terms ("40 more points and your stake drops to 1.5×"). A default entry says "This stays on your record" at the moment it is written. Members never see other members' scores.

**States and error handling**

- **UX-DR16:** Implement the 19 specified state treatments from `EXPERIENCE.md § State Patterns`, including: insufficient balance (exact dollar shortfall broken into stake + first contribution), room filling and room cancelled, grace window (mine) and grace window (another member's, blame-free), default resolved, removed via stake exhaustion, payout received (no celebration), cycle complete, transaction submitted, transaction failed, wallet session expired, and offline with an "as of" timestamp.
- **UX-DR17:** On transaction failure, state the plain cause where knowable and make an unambiguous statement of whether money moved. Read-only surfaces stay readable when a wallet session expires; stale money state is never shown without its timestamp.

**Interaction and wallet round trip**

- **UX-DR18:** Design the WalletConnect signing handoff as an explicit flow: state what will be signed and its amount before leaving, resolve to a definite state on return (succeeded / rejected / failed), and reconstruct pending transaction state from chain state rather than memory so it survives iOS Safari backgrounding.
- **UX-DR19:** Tap to act only — no long-press, swipe actions, or meaningful gestures. Pull-to-refresh on Room home and Underwriter room only. One primary action per surface. Banned outright: confirmation dialogs over commit screens, toasts carrying consequential information, countdown timers as the sole expression of a deadline, carousels, onboarding tours, celebratory animation, badge counts, streaks, and re-engagement prompts.

**Voice and copy**

- **UX-DR20:** Enforce the microcopy rules: plain language and honest mechanics, never volunteering "stablecoin", "smart contract", "escrow", or "blockchain" while never implying money is somewhere it isn't or softening an irreversible consequence. Sentence case, full sentences, no exclamation marks, exact amounts in commitment copy, and the underwriter named as a person.

**Accessibility**

- **UX-DR21:** Never encode a money state in color alone — all four states carry their meaning in text. Tap targets ≥ 48dp with the primary funding action at 52px minimum. Text scales to 200% without truncating any amount, date, or control label (the schedule is the test case). Focus traversal follows reading order with the commit control always last, after its disclosure.
- **UX-DR22:** The grace banner is an `aria-live="assertive"` region; Room home state changes announce; amounts are announced as currency, not digit strings. Contrast floor is WCAG AA overall, with the FX disclosure and grace banner held to AAA body text.
- **UX-DR23:** No motion carries meaning; `prefers-reduced-motion` degrades to no visible change.

**Responsive and platform**

- **UX-DR24:** Mobile-first single column at every width, content max-width ~560px on larger viewports — the layout gains margin, never a second column. Android Chrome is the primary target; iOS Safari is supported; PWA install is offered but no functionality is gated behind it. Underwriter surfaces gain rows and wider tables at desktop width without becoming desktop-only.

**Notification (constrained, with a named gap)**

- **UX-DR25:** V1 ships in-app notification only. FR-14's two required notifications are delivered as Room home state plus a persistent, non-dismissible grace banner that survives navigation across every surface, not only Room home. Round deadlines are stated as absolute dates and times, never as relative countdowns alone. `[ASSUMPTION from spine: an "add to calendar" affordance on the schedule is the cheapest external reach available — worth building if the window allows.]` SMS escalation at grace-open and 24-hours-remaining is a recorded deferral to switch on before the mainnet pilot opens with real value.

### FR Coverage Map

| FR | Epic | Coverage |
|---|---|---|
| FR-1 | Epic 1 | Member identity verification through the KYC Provider; attestation held by Registry |
| FR-2 | Epic 1 | Verification tier writes the starting Trust Score to Registry |
| FR-3 | Epic 1 | Duplicate detection enforced at attestation issue; duplicate-flagged join reverts |
| FR-4 | Epic 2 | Room creation through the Factory with cadence and capacity validation |
| FR-5 | Epic 2 | Applicant review and admission on Pal3 history only |
| FR-6 | Epic 2 | Automatic Room start at member count; parameter immutability; open-window cancellation |
| FR-7 | Epic 3 | Room close after the final Round, returning remaining Stakes |
| FR-8 | Epic 3 | Scheduled Contribution into Room escrow |
| FR-9 | Epic 3 | Per-Member contribution status by handle for the current Round |
| FR-10 | Epic 4 | Event-sourced Trust Score computation and Registry commit |
| FR-11 | Epic 2 | Payout ordering with Fairness Floor, computed on-chain and frozen at Room start |
| FR-12 | Epic 3 | Full-Pot payout on the scheduled Round boundary |
| FR-13 | Epic 2 | Stake sizing from the snapshotted Trust Score, displayed before commitment |
| FR-14 | Epic 3 | 48-hour Grace Window, notifications, and cured contribution |
| FR-15 | Epic 3 | Atomic Slash → Backstop → Payout waterfall |
| FR-16 | Epic 3 | Stake exhaustion, Member removal, Backstop carry of remaining Rounds |
| FR-17 | Epic 3 | Underwriter live and maximum exposure visibility |
| FR-18 | Epic 2 | Tier capacity limits enforced at Room creation |
| FR-19 | Epic 2 | Capital adequacy gate — Backstop posted and locked before admission opens |
| FR-20 | Epic 3 | Per-Contribution Underwriter fee, disclosed pre-join and charged on Contribution |
| FR-21 | Epic 1 | WalletConnect wallet connection and balance/shortfall reporting |
| FR-22 | Epic 2 | Complete Room terms on one screen with recorded FX acknowledgement |
| FR-23 | Epic 3 + Epic 4 | Schedule and Room state (Epic 3); Trust Score and its change history (Epic 4) |

All 23 FRs mapped. Distribution: Epic 1 → 4 FRs, Epic 2 → 8, Epic 3 → 10, Epic 4 → 1 (plus half of FR-23), Epic 5 → 0 (NFR/AR gate epic).

## Epic List

### Epic 1: A verified member with a working wallet

A person can install the PWA, connect a Stellar wallet from their phone, verify their identity through the KYC Provider, and see the Trust Score that verification earned them — with Pal3 holding no identity documents at any point.

**FRs covered:** FR-1, FR-2, FR-3, FR-21
**NFRs/ARs carried:** NFR-1 (test harness), NFR-2, NFR-3, NFR-6, NFR-7, NFR-10, AR-2 (Registry), AR-3, AR-4, AR-6, AR-8, AR-9, AR-10, AR-11, AR-15, AR-16, AR-18, AR-19, AR-20, AR-23
**UX-DRs carried:** UX-DR1, UX-DR2, UX-DR3, UX-DR4, UX-DR6 (partial), UX-DR8 (Welcome, Connect wallet, Verify identity, Verification status, Account, Activity), UX-DR9, UX-DR18, UX-DR20, UX-DR21, UX-DR24
**Standalone:** yes — a verified, wallet-connected identity is complete and useful before any Room exists.
**Notes:** Carries the scaffold story — source tree, pinned toolchain, generated bindings (no starter template exists; the Architecture specifies a toolchain and a layout). Builds `contracts/registry` and `services/kyc`. AR-15 is exercised here first: every entry point authorizes the transaction source only.

### Epic 2: A room you can read completely before you commit

An Underwriter opens a Room, posts Backstop, and admits Members on Pal3 history alone. A Member sees the complete terms — contribution, cadence, their Payout Position and date, their Stake as multiple and amount, the fee, and the default waterfall in plain language — and commits once. The Room fills, starts, and freezes.

**FRs covered:** FR-4, FR-5, FR-6, FR-11, FR-13, FR-18, FR-19, FR-22
**NFRs/ARs carried:** NFR-2, NFR-3, NFR-8, AR-2 (Factory + Room creation), AR-4, AR-5, AR-6, AR-11, AR-14, AR-17, AR-18
**UX-DRs carried:** UX-DR5, UX-DR6, UX-DR8 (Room terms, Funding check, Commit, Room filling, Create room, Post backstop, Applicants), UX-DR10, UX-DR11, UX-DR12, UX-DR13, UX-DR14, UX-DR16 (insufficient balance, room filling, room cancelled)
**Standalone:** yes for development — it delivers complete functionality for its domain and requires no later epic to build. **Not independently deployable with real value:** a Room that has formed and frozen holds Member Stakes and first Contributions with no path to run Rounds or return funds except open-window cancellation. This epic must not reach mainnet ahead of Epic 3.
**Notes:** FR-11 sits here rather than with payouts because ordering is computed *at Room start* (AD-5) and is visible before it. The join-time score snapshot with revert-on-mismatch is the AD-5 story.

### Epic 3: A cycle that pays out on schedule — and covers a miss without anyone losing money

The Room runs. Members contribute, see each other's status, and watch the schedule advance. Each Round's recipient receives the full Pot on the scheduled date — including when someone misses, through Grace Window, Slash, and Backstop. The Room closes and returns Stakes.

**FRs covered:** FR-7, FR-8, FR-9, FR-12, FR-14, FR-15, FR-16, FR-17, FR-20, FR-23 (schedule and Room state)
**NFRs/ARs carried:** NFR-2, NFR-3, NFR-4, NFR-5, AR-1, AR-3, AR-4, AR-6, AR-9, AR-13, AR-17, AR-18, AR-21, AR-24
**UX-DRs carried:** UX-DR6, UX-DR8 (Room home, Contribute, Members, Round detail, Cycle summary, Underwriter room), UX-DR11, UX-DR12, UX-DR16, UX-DR17, UX-DR19, UX-DR25
**Standalone:** yes, and deliberately large.
**Notes:** Kept whole rather than split from the waterfall. Separating "the cycle runs" from "the waterfall" would ship an intermediate Room that pays out only if every Member pays — not a shippable increment for a product whose central claim is that the recipient is paid in full in every branch. AD-9 additionally requires Slash, Backstop draw, and Payout to resolve in a single atomic invocation, so a split would mean authoring `advance_round` twice. Also carries the indexer and API read models, first needed here for whole-cycle views.

### Epic 4: Trust that compounds

A Member's score moves on what they actually did, they can recite why it moved, and the score they end a Cycle with visibly changes what their next Room costs them.

**FRs covered:** FR-10, FR-23 (Trust Score and change history)
**NFRs/ARs carried:** NFR-2, NFR-8, NFR-11, AR-7, AR-11, AR-21, AR-22
**UX-DRs carried:** UX-DR6 (trust score panel), UX-DR7, UX-DR8 (Trust score, Cycle summary), UX-DR15, UX-DR20
**Standalone:** yes — depends only on Epic 1's Registry, not on Epic 2 or Epic 3.
**Notes:** One FR, but a substantial one: `services/trust` as a versioned pure function over chain event history, committing to Registry only between Rooms (AR-7), plus the Trust surface and the cycle-summary carry-forward. This is the epic SM-5 measures directly, and it can be built in parallel with Epics 2–3.

### Epic 5: Pilot readiness — testnet to mainnet

The contracts are public, audit-ready, and deployed along a path where no Room holds real value until the audit-readiness review has passed.

**FRs covered:** none — this epic carries non-functional and architectural gates.
**NFRs/ARs carried:** NFR-1, NFR-2, NFR-3, NFR-5, NFR-9, AR-12, AR-25
**Standalone:** yes — a gate, not a feature.
**Notes:** Included as an epic rather than a checklist because NFR-1 is an absolute precondition ("no mainnet pilot begins without it") and open-sourcing is part of the trust proposition rather than a release chore.

### Dependencies

Epic 1 → Epic 2 → Epic 3 is a genuine chain: Registry before Rooms, a formed Room before a running Cycle. Epic 4 branches off Epic 1 and is independent of Epics 2 and 3. Epic 5 gates mainnet, not development.

## Epic 1: A verified member with a working wallet

A person can install the PWA, connect a Stellar wallet from their phone, verify their identity through the KYC Provider, and see the Trust Score that verification earned them — with Pal3 holding no identity documents at any point.

### Story 1.1: Project scaffold with generated contract bindings

As a developer on Pal3,
I want the repository scaffolded to the architecture's source tree with a working contract build and binding generation,
So that every subsequent story starts from a layout and toolchain that cannot drift from the contracts.

**Acceptance Criteria:**

**Given** an empty repository
**When** the scaffold is created
**Then** the tree contains `contracts/` (`registry`, `factory`, `room`, `shared`), `services/` (`trust`, `kyc`, `indexer`, `rates`, `api`), `app/`, `shared/`, and `bindings/` exactly as the Architecture Spine specifies
**And** `app/` is scaffolded as React + Vite with `vite-plugin-pwa` supplying the service worker and manifest, and `shared/` holds the single implementation of cross-cutting derivations consumed by both `app/` and `services/` (AD-17)
**And** `contracts/shared` holds the Glossary-named common types and one `#[contracterror]` enum per contract with explicit stable discriminants (AR-16, AR-20).

**Given** the pinned toolchain
**When** the contract workspace builds
**Then** it builds with `soroban-sdk` 25.0.0 on Rust 1.84+ targeting `wasm32v1-none`
**And** the installed `stellar-cli`, `@stellar/stellar-sdk`, and Stellar Wallets Kit versions are recorded in the repository (AR-8, Stack table).

**Given** a successful contract build
**When** the build completes
**Then** TypeScript clients are regenerated into `bindings/` by `stellar contract bindings` as part of the build, not as a manual step
**And** `bindings/` is marked generated and excluded from hand-editing by lint or CI check (AR-8).

**Given** any service or client source file
**When** it is inspected for configuration
**Then** contract addresses and the network passphrase are read from environment at build/deploy time
**And** no contract address or passphrase literal appears in source (AR-23).

**Given** the contract source
**When** the repository is published
**Then** every crate under `contracts/` carries an Apache 2.0 licence header and the repository carries the Apache 2.0 licence file (AR-12, NFR-9).

### Story 1.2: Contract test harness and CI pipeline

As a Member whose savings these contracts will hold,
I want every change to the contracts built, verified, and tested automatically,
So that the code holding my money is never changed without something checking it.

**Acceptance Criteria:**

**Given** the contract workspace
**When** the test harness is established
**Then** Soroban contract unit tests run against each contract using the SDK's test utilities, with conventions documented for asserting reverts by error code rather than by message (AR-20).

**Given** a Room under test
**When** an integration test exercises a full Cycle
**Then** the harness can form a Room, run every Round, induce a Default, resolve it through the waterfall, and close — with assertions available at each transition (SM-1, SM-2, SM-3).

**Given** a test that advances time
**When** it manipulates the clock
**Then** it advances the ledger timestamp rather than a wall clock, so Grace Window and Round deadline behavior is testable deterministically (AR-4, AR-18).

**Given** any change pushed to the repository
**When** CI runs
**Then** it builds the contract workspace, regenerates TypeScript bindings, fails if regenerated bindings differ from committed ones, and runs the full test suite (AR-8).

**Given** CI
**When** it runs against the client and services
**Then** it type-checks and tests them against the generated bindings rather than hand-written contract types (AR-8).

**Given** a test suite run
**When** it completes
**Then** the result is deterministic — the same commit produces the same outcome, with no randomness or wall-clock dependence in any test (NFR-2, AR-11).

**Given** the invariant properties that Story 5.3 asserts
**When** the harness is designed
**Then** it can express them as tests — no privileged transfer path, atomic waterfall revert, no randomness, expiry never as control flow — rather than leaving them to manual review (NFR-1, NFR-3, AR-6, AR-9).

### Story 1.3: Design token foundation and PWA shell

As a member on a mid-range Android phone,
I want the app to render legibly and consistently from the first screen,
So that I can read amounts and decisions clearly before I am ever asked to commit money.

**Acceptance Criteria:**

**Given** the design token set in `DESIGN.md`
**When** the app's styling layer is implemented
**Then** all 16 color tokens, 8 typography roles, 5 radii, and the full spacing scale including `margin-mobile`, `gutter`, and `commit-gap` exist as the single source of styling truth
**And** no hard-coded color, font size, or spacing literal appears in component source (UX-DR1).

**Given** the money-state color rules
**When** a lint or review check runs over the styling layer
**Then** `accent` is used only for commitment, trust-score fill, and current-round marking
**And** `state-paid`, `state-grace`, and `state-default` appear only on money-state elements, and the two wash colors only behind the grace banner and a defaulted row (UX-DR2).

**Given** a monetary figure anywhere in the app
**When** it renders
**Then** it uses the `amount` or `amount-inline` role with `tabular-nums`
**And** the layout survives the browser font-size setting raised to 200% without truncating any amount, date, or control label (UX-DR3, UX-DR21).

**Given** the visual system
**When** any surface renders
**Then** no shadow, gradient, illustration, or floating card appears; raised surfaces are distinguished by tone plus a hairline border
**And** a scrim on sheets and modals is the only depth cue present (UX-DR4).

**Given** a viewport of any width
**When** a surface renders
**Then** content is a single column capped near 560px, gaining margin rather than a second column
**And** the app installs as a PWA on Android Chrome and iOS Safari with no functionality gated behind installing (UX-DR24, NFR-10).

**Given** the measured-contrast table in `DESIGN.md § Colors`
**When** the palette is implemented
**Then** every load-bearing pair meets its stated ratio, verified by measurement rather than by eye
**And** the grace banner and FX disclosure meet AAA body text, since both are read by a Member in poor conditions at the moment money is at stake (UX-DR1, UX-DR22).

**Given** no component library underpins the accessibility floor
**When** focus traversal, ARIA labelling, dialog semantics, and the `aria-live` grace banner are built
**Then** each is implemented and verified explicitly rather than inherited from a primitive's default behavior (UX-DR21, UX-DR22, `EXPERIENCE.md § Foundation`).

**Given** any animation or transition in the app
**When** it is reviewed
**Then** no motion carries meaning — nothing in the product requires perceiving an animation to understand state
**And** `prefers-reduced-motion` degrades to no visible change (UX-DR23).

**Given** the primitives this epic needs
**When** they are built
**Then** primary button (52px minimum height, one per surface), secondary button, and disclosure block exist to their `DESIGN.md` specs
**And** every interactive element carries a role and state label, tap targets are ≥ 48dp, and focus traversal follows reading order (UX-DR6 partial, UX-DR21).

### Story 1.4: Registry attestations with duplicate rejection

As Pal3,
I want the Registry contract to hold an opaque KYC Attestation per member address and reject duplicates,
So that a real, unique, verified person stands behind every wallet that will ever touch a Room.

**Acceptance Criteria:**

**Given** the Registry contract is deployed with an admin authority
**When** an attestation is issued for a member address
**Then** the stored record contains only: attestation status, verification tier, issue date, and an opaque provider reference string
**And** no name, document, employer, bank detail, or hash derived from a raw identity document is written on-chain (AR-10, FR-1, NFR-7).

**Given** a stored attestation
**When** its keying and references are examined
**Then** the Member is keyed by their Stellar address, and the provider reference is an opaque string rather than a value derived from any identity document (AR-19).

**Given** an address that already holds an attestation whose provider reference maps to the same individual
**When** a second attestation issue is attempted
**Then** the invocation reverts with the duplicate error code
**And** the original attestation is unchanged (FR-3).

**Given** an attestation flagged as duplicate by the provider
**When** its status is queried
**Then** the Registry reports the flag, so any downstream join check can revert on it (FR-3).

**Given** an attestation whose validity period has passed
**When** its status is queried
**Then** the Registry reports it as expired rather than deleting it
**And** the record remains readable for an in-progress Room (FR-1).

**Given** an address without the admin authority
**When** it attempts to issue, alter, or revoke an attestation
**Then** the invocation reverts
**And** the Registry exposes no entry point by which admin authority could move value in any contract (AR-6).

**Given** any Registry entry point
**When** it is invoked
**Then** authorization is a single `require_auth` on the transaction source, with no detached auth entry required (AR-15)
**And** attestation state uses `persistent` storage with its TTL extended on every write (AR-3).

**Given** any attestation issue, revoke, or expiry transition
**When** it completes
**Then** the contract emits an event carrying the address and the transition (AR-21).

### Story 1.5: Starting Trust Score from verification tier

As a newly verified member,
I want my verification tier to give me a starting Trust Score,
So that I begin with a reputation I earned by proving who I am, before I have completed any cycle.

**Acceptance Criteria:**

**Given** a member whose attestation is issued at basic tier (government ID only)
**When** their starting Trust Score is written
**Then** the Registry records 300 (FR-2).

**Given** a member whose attestation is issued at full tier (ID, employment record, and bank detail)
**When** their starting Trust Score is written
**Then** the Registry records 400 (FR-2).

**Given** a member who has completed zero Cycles
**When** any Trust Score write above 400 is attempted for them
**Then** the write is rejected and the score remains at or below 400 (FR-2).

**Given** any Trust Score write
**When** it is applied
**Then** the resulting value is clamped to the range 0–1000 (FR-10 clamp rule)
**And** the write emits an event carrying the address, the previous value, and the new value (AR-21).

**Given** a Trust Score entry
**When** it is stored
**Then** it uses `persistent` storage with TTL extended on write, and holds an integer only (AR-3, AR-10).

**Given** an address without the Registry admin authority
**When** it attempts to write a Trust Score
**Then** the invocation reverts (AR-6).

### Story 1.6: KYC provider webhook issues attestations

As Pal3,
I want the KYC adapter to turn a provider verification result into a Registry attestation,
So that verification completed in the provider's flow becomes an on-chain fact without Pal3 ever handling documents.

**Acceptance Criteria:**

**Given** a verification-approved webhook from the KYC Provider carrying a provider reference, tier, and the member's address
**When** the adapter processes it
**Then** it submits an attestation issue to Registry at the reported tier and the corresponding starting Trust Score is written (FR-1, FR-2).

**Given** any webhook payload containing identity fields
**When** the adapter processes it
**Then** only the provider reference, tier, status, and address are retained or forwarded
**And** no document image, name, employment record, or bank detail is persisted by Pal3 in any store or log (FR-1, NFR-6, NFR-7).

**Given** a webhook reporting a duplicate individual
**When** the adapter processes it
**Then** no attestation is issued and the outcome is recorded as duplicate for the verification-status surface (FR-3).

**Given** a webhook reporting verification failure
**When** the adapter processes it
**Then** no attestation is issued and the outcome and the provider's stated remedy path are available to the verification-status surface (FR-1).

**Given** a webhook that cannot be authenticated as originating from the KYC Provider
**When** it arrives
**Then** it is rejected without any Registry submission.

**Given** the same webhook delivered more than once
**When** the adapter processes it
**Then** the result is identical to a single delivery and no second attestation is issued.

**Given** an active Cycle is running in any Room
**When** the adapter has work to submit
**Then** it writes to Registry only, never to a Room contract (AR-7).

### Story 1.7: Wallet connection over WalletConnect

As a member on my phone,
I want to connect my Stellar wallet and see my balance,
So that the app knows who I am on-chain and I can see what I have before anything asks me to commit it.

**Acceptance Criteria:**

**Given** an unauthenticated visitor opening the app
**When** the Welcome surface renders
**Then** it states what Pal3 is in four lines with one action — connect a wallet — and no marketing scroll (UX-DR8).

**Given** a member on Android Chrome or iOS Safari
**When** they connect through WalletConnect
**Then** a session is established with at least one WalletConnect-compatible Stellar wallet end to end and the app holds their address (FR-21).

**Given** an established session
**When** the app reads the member's balance of the Room stablecoin asset
**Then** the balance is displayed using the amount display component, with the dollar figure as the number of record (FR-21, UX-DR10).

**Given** a member leaving the app to sign in their wallet
**When** the handoff occurs
**Then** the app states what will be signed and its amount before leaving
**And** on return it resolves to a definite state — succeeded, rejected, or failed — never leaving an ambiguous screen (UX-DR18).

**Given** a member returning after iOS Safari discarded the background tab
**When** the app reloads
**Then** pending transaction state is reconstructed from chain state rather than from memory (UX-DR18).

**Given** an expired or dropped wallet session
**When** the member is on any surface
**Then** a reconnect prompt appears in place
**And** read-only surfaces remain readable (UX-DR16).

**Given** the app is offline
**When** any surface renders
**Then** last-known state is shown with an "as of" timestamp and signing is disabled with the reason stated (UX-DR16, UX-DR17).

**Given** a connected member whose address underwrites a Room
**When** the app resolves their role
**Then** the Underwrite tab is revealed; otherwise the member sees Room, Trust, and Account only, with no drawer and sheets stacking one level deep (UX-DR9).

### Story 1.8: Identity verification handoff and status

As a prospective member,
I want to verify my identity through a licensed provider and see exactly what it earned me,
So that I know Pal3 never holds my documents and I know the score I am starting from.

**Acceptance Criteria:**

**Given** a connected wallet with no attestation
**When** the Verify identity surface renders
**Then** it states what verification is for, that a licensed provider holds the documents, and what Pal3 never sees
**And** it hands off to the provider's flow (FR-1, UX-DR8).

**Given** a member returning from the provider with verification approved
**When** the Verification status surface renders
**Then** it shows the verified tier, the resulting starting Trust Score, and what that score buys stated as a stake multiple (FR-2, UX-DR15).

**Given** verification is still pending at the provider
**When** the Verification status surface renders
**Then** it states that the provider is working, shows no countdown Pal3 cannot honor, leaves the app usable, and states the reason joining is blocked (UX-DR16).

**Given** verification failed
**When** the Verification status surface renders
**Then** it states the outcome and the provider's remedy path
**And** does not imply that Pal3 offers an appeal, because none exists (FR-1, UX-DR16).

**Given** verification returned a duplicate result
**When** the Verification status surface renders
**Then** it states the duplicate outcome and the provider's remedy path (FR-3, UX-DR16).

**Given** any copy on these surfaces
**When** it is reviewed
**Then** it is sentence case, in full sentences, free of exclamation marks, and never volunteers "stablecoin", "smart contract", "escrow", or "blockchain" (UX-DR20).

**Given** a member with no valid attestation
**When** any join path is attempted
**Then** it is blocked with the reason stated in text, not in a tooltip (FR-1, UX-DR14).

### Story 1.9: Account and Activity

As a member,
I want to see my handle, verification tier, and every on-chain action my address has taken,
So that my wallet session and my irreversible history both have a place I can inspect.

**Acceptance Criteria:**

**Given** a connected, verified member
**When** the Account surface renders
**Then** it shows their handle, verification tier, and wallet session, with a disconnect action (FR-1, UX-DR8).

**Given** a member on the Account surface
**When** they open Activity
**Then** every on-chain action taken by their address is listed with an explorer link for each (UX-DR8).

**Given** the Members-facing surfaces in this epic
**When** any member is represented
**Then** they are shown by handle only, with no avatar, name, or photo (UX-DR6).

**Given** a Member's Stellar address
**When** their handle is rendered anywhere in the app
**Then** it is produced by the single shared derivation module, rendered in the `handle` typography role with tabular figures, grouped, drawn from an alphabet excluding `0`/`O` and `1`/`I`, and announced character by character by screen readers (AD-17, UX-DR26).

**Given** a Member who has taken no on-chain action yet
**When** Activity renders
**Then** it states in one line that actions appear here once taken, with no placeholder rows and no illustration (UX-DR27).

**Given** a member who disconnects
**When** the session ends
**Then** no attestation, score, or on-chain state is altered, and reconnecting the same address restores the same view (AR-1).

## Epic 2: A room you can read completely before you commit

An Underwriter opens a Room, posts Backstop, and admits Members on Pal3 history alone. A Member sees the complete terms — contribution, cadence, their Payout Position and date, their Stake as multiple and amount, the fee, and the default waterfall in plain language — and commits once. The Room fills, starts, and freezes.

> **Spec tension recorded 2026-08-12.** FR-22 requires the pre-commit terms screen to show the Member's Payout Position, but FR-11 computes positions only at Room start when the Room is full. A Member joining a partially filled Room can only be shown a *provisional* position, because a later joiner with a higher Trust Score displaces them. Story 2.8 therefore specifies a provisional position with the ranking rule stated in plain language, and the frozen position appears only after start (Story 2.6). Resolving this any other way requires either weakening FR-22's disclosure or breaking FR-11's freeze.

### Story 2.1: Factory deploys a Room with validated parameters

As an Underwriter,
I want to create a Room with a fixed Contribution, Cadence, and Member count that the Factory validates against my tier,
So that a Room can never exist with parameters outside what my capacity permits.

**Acceptance Criteria:**

**Given** an Underwriter within their subscription tier
**When** they invoke Room creation with a Contribution amount, weekly Cadence, and Member count
**Then** the Factory deploys a Room instance holding those parameters, in a created state that does not yet admit Members (FR-4, AR-2).

**Given** a Member count above the tier maximum or a concurrent-Room count at the tier limit
**When** Room creation is invoked
**Then** the invocation reverts with the capacity error code and no Room is deployed (FR-18).

**Given** V1's single tier
**When** capacity is evaluated
**Then** the limits enforced are one concurrent Room and a maximum of 10 Members
**And** a Member count below the pilot minimum of 5 also reverts (FR-18, PRD §6.1).

**Given** a Cadence other than weekly
**When** Room creation is invoked
**Then** the invocation reverts (FR-4).

**Given** a deployed Room
**When** its parameters are stored
**Then** Contribution is an integer stroop amount, Cadence and the Grace Window are second offsets, and no floating point or client-supplied time is accepted (AR-17, AR-18).

**Given** any deployed Room
**When** its state is inspected
**Then** it holds no reference to any other Room, and Registry holds no reference to it (AR-2)
**And** all Room state uses `persistent` storage with TTL extended on every write (AR-3).

**Given** Room creation completes
**When** the transaction settles
**Then** the Factory emits a Room-created event carrying the Room address, Underwriter, Contribution, Cadence, and Member count (AR-21).

### Story 2.2: Backstop posting gates Member admission

As a Member considering a Room,
I want admission to be impossible until the Underwriter's Backstop is posted and locked in the Room itself,
So that the capital covering a default is already there before I put anything in.

**Acceptance Criteria:**

**Given** a created Room not yet admitting Members
**When** the required Backstop is computed
**Then** it equals at least 2 × Contribution × Member count and is readable from the Room before anyone commits (FR-19).

**Given** an Underwriter posting Backstop
**When** the transaction succeeds
**Then** the capital is transferred into the Room contract itself, not held by the Factory
**And** the Room transitions to admitting Members (FR-19, AR-14).

**Given** a Backstop transfer below the required amount
**When** it is submitted
**Then** the invocation reverts and the Room does not open admission (FR-19).

**Given** a Room in progress that the Backstop collateralizes
**When** the Underwriter attempts to withdraw Backstop capital
**Then** the invocation reverts (FR-19).

**Given** posted Backstop capital held by the Room
**When** the entry points that can move it are enumerated
**Then** the only paths are the waterfall draw and the Room-close return, and no other path exists by which it can leave the Room (AR-14, AR-6).

**Given** any address including the Underwriter and any operator
**When** it attempts to move Backstop capital outside the waterfall or the close path
**Then** no entry point exists that permits it (AR-6, NFR-3).

### Story 2.3: Underwriter creates a Room and posts Backstop

As an Underwriter,
I want to configure a Room and see my maximum exposure and required Backstop before I sign,
So that I know the most I can lose before I commit any capital.

**Acceptance Criteria:**

**Given** an address the app resolves as an Underwriter
**When** the app loads
**Then** the Underwrite tab is present and the Create room surface is reachable from it (UX-DR9, UX-DR8).

**Given** the Create room surface
**When** the Underwriter sets Contribution, Cadence, and Member count
**Then** the inputs are bounded by their tier and a value outside it is refused with the limit stated in text (FR-18, UX-DR14).

**Given** configured Room parameters
**When** the surface renders before any signature
**Then** it states the maximum possible exposure for the Room as an amount (FR-17 pre-commit half)
**And** it states the required Backstop as both the formula and the resulting figure (FR-19).

**Given** the Post backstop surface
**When** it renders
**Then** it follows the commit pattern: what is given up as an amount, what it covers, what happens if it is drawn, `commit-gap`, then one primary action labeled with act and amount (UX-DR5, UX-DR12).

**Given** the Underwriter signs the Backstop transaction
**When** it settles
**Then** the surface states that the capital is locked and cannot be withdrawn while the Room runs, and admission is now open (FR-19, UX-DR20).

**Given** the transaction is rejected in the wallet or fails on chain
**When** the app resolves the outcome
**Then** it states the plain cause and makes an unambiguous statement of whether money moved (UX-DR17).

### Story 2.4: Applicant review and admission on Pal3 history

As an Underwriter,
I want to admit or reject applicants on their Pal3 record alone,
So that I can price the risk I am taking without ever seeing who they are.

**Acceptance Criteria:**

**Given** a Room admitting Members and a verified applicant
**When** the Underwriter opens the Applicants surface
**Then** each request shows Trust Score, Cycles completed, Default count, and account age, by handle derived from the shared module (FR-5, AD-17, UX-DR26).

**Given** a Room with admission open and no requests yet
**When** the Applicants surface renders
**Then** it states that admission is open and how many Members the Room still needs, rather than an empty-results message (UX-DR27).

**Given** the Applicants surface
**When** it renders any applicant
**Then** it shows no name, document, employer, bank detail, avatar, or photo (FR-5, UX-DR6, NFR-7).

**Given** an applicant the Underwriter admits
**When** the admission invocation succeeds
**Then** that address becomes eligible to join the Room and the decision emits an event (FR-5, AR-21).

**Given** an applicant the Underwriter rejects
**When** the rejection settles
**Then** any funds that applicant had locked for this Room are returned in full to their address (FR-5).

**Given** an address that is not the Room's Underwriter
**When** it attempts to admit or reject
**Then** the invocation reverts (AR-6, AR-15).

**Given** admission is an irreversible act that fixes a Member into a Room that becomes immutable at start
**When** the Underwriter admits
**Then** the surface follows the commit pattern with the consequence stated before the action (UX-DR12, UX-DR13).

### Story 2.5: Member join with score snapshot and Stake lock

As a Member,
I want joining to lock my Stake and first Contribution in one signed transaction at the terms I was shown,
So that nothing about my commitment can change between reading the screen and signing it.

**Acceptance Criteria:**

**Given** an admitted Member with a valid attestation
**When** they invoke join
**Then** the Room snapshots their Trust Score from Registry into its own state, and that snapshot is immutable for the Room's lifetime (AR-5).

**Given** a join transaction carrying the Trust Score value the Member was shown
**When** the Registry value no longer matches it
**Then** the invocation reverts (AR-5).

**Given** a snapshotted Trust Score
**When** the required Stake is computed
**Then** Stake = Contribution × (2 − TrustScore/1000), yielding 2× Contribution at score 0 and 1× at score 1000, as integer stroops (FR-13, AR-17).

**Given** a successful join
**When** the transaction settles
**Then** the Stake and the first Contribution are both locked in the Room in that single transaction
**And** the Stake is not recomputed for the remainder of the Cycle even if the Member's Registry score changes (FR-13).

**Given** a wallet with no valid attestation, an expired attestation, or an attestation flagged duplicate
**When** join is invoked
**Then** the invocation reverts (FR-1, FR-3).

**Given** an address that has not been admitted by the Underwriter
**When** join is invoked
**Then** the invocation reverts (FR-5).

**Given** a Room already at its configured Member count
**When** a further join is invoked
**Then** the invocation reverts (FR-6).

**Given** any join entry point
**When** it is invoked
**Then** authorization is a single `require_auth` on the transaction source with no detached auth entry, and no second party signs within the invocation (AR-15).

**Given** a successful join
**When** it settles
**Then** the Room emits a join event carrying the address, snapshotted score, Stake amount, and Contribution amount (AR-21).

### Story 2.6: Room start with Fairness Floor ordering and frozen parameters

As a Member,
I want payout positions computed on-chain the moment the room fills and frozen thereafter,
So that a date I can plan around comes from a contract that cannot decide otherwise.

**Acceptance Criteria:**

**Given** a Room that has reached its configured Member count
**When** the start transition is invoked
**Then** the Room starts, and it starts at no other time and for no other reason (FR-6).

**Given** a started Room
**When** Payout Positions are computed
**Then** the Room computes them itself from its own Trust Score snapshots, ranked descending, with the highest score receiving the Pot in Round 1
**And** no off-chain component submits or influences the ordering (FR-11, AR-5).

**Given** two Members with equal snapshotted Trust Scores
**When** ordering is computed
**Then** the tie is broken by the earlier Room-join timestamp, with no randomness anywhere in the computation (FR-11, AR-11, NFR-2).

**Given** a Member who held the final Payout Position in the two immediately preceding Cycles within this same Room
**When** ordering is computed
**Then** they are not assigned it a third consecutive time and instead swap with the next-lowest-ranked eligible Member (FR-11).

**Given** computed positions
**When** the Room has started
**Then** the full ordering and every Round's date are visible to every Member
**And** no position may change for the remainder of the Cycle (FR-11, FR-23).

**Given** a started Room
**When** any change to Contribution amount, Cadence, ordering, or membership is attempted
**Then** it reverts — the only permitted membership change is removal through Stake exhaustion (FR-6, NFR-3).

**Given** the same set of snapshots and join timestamps
**When** ordering is computed by any independent observer from chain history
**Then** the result is identical (NFR-2, AR-11).

**Given** Room start completes
**When** the transaction settles
**Then** the Room emits a start event carrying every Member address, their position, and each Round's deadline (AR-21).

### Story 2.7: Open-window cancellation returns everything in full

As a Member who joined a room that never filled,
I want my Stake and Contribution returned in full,
So that committing early to a room that did not happen costs me nothing.

**Acceptance Criteria:**

**Given** a Room admitting Members
**When** it is created
**Then** an open-window deadline of 14 days is stored in Room state as a ledger-timestamp deadline (FR-6, AR-4, AR-18).

**Given** the ledger timestamp has passed the open-window deadline and the Room has not reached its Member count
**When** the cancellation transition is invoked
**Then** the Room cancels and every joined Member's Stake and Contribution are returned in full to their address (FR-6).

**Given** a cancelled Room
**When** the Underwriter's position is settled
**Then** the posted Backstop returns to the Underwriter in full (AR-14).

**Given** the open window has not yet elapsed
**When** cancellation is invoked
**Then** it reverts (AR-4).

**Given** the cancellation transition
**When** any address invokes it after the deadline
**Then** it succeeds identically regardless of caller and is idempotent on repeat invocation (AR-13).

**Given** a Room whose entries are approaching TTL expiry
**When** any transition is evaluated
**Then** no branch depends on whether an entry has expired; only the stored deadline against ledger timestamp governs (AR-4).

### Story 2.8: Room terms disclosure with FX acknowledgement

As a prospective Member,
I want every term of the room on one screen before I commit anything,
So that I am never surprised by something I could have been told first.

**Acceptance Criteria:**

**Given** an admitted Member viewing a Room
**When** the Room terms surface renders
**Then** one screen shows: Contribution amount, Cadence, Member count, their Payout Position and date, their Stake, the Underwriter's fee, and the default-handling waterfall in plain language (FR-22).

**Given** the Room has not yet started
**When** the Payout Position is shown
**Then** it is presented as provisional with the ranking rule stated plainly — that a Member joining with a higher Trust Score moves them later, and that the position is final when the Room fills (FR-11, FR-22, UX-DR20).

**Given** the Stake is displayed
**When** it renders
**Then** it shows as multiple and amount together, alongside the Trust Score that produced it (FR-13, UX-DR11, UX-DR15).

**Given** the Underwriter's fee is displayed
**When** it renders
**Then** it shows as both a percentage and an absolute per-Round amount, and states that it is charged on Contribution so the Pot is never reduced (FR-20, UX-DR11).

**Given** the Pot is displayed
**When** it renders
**Then** it shows its full undeducted value, and no "net" payout figure appears anywhere on the surface (UX-DR11).

**Given** any peso figure on this surface
**When** it renders
**Then** it appears beneath the dollar figure in `meta` prefixed `≈`, never inside a button label
**And** attribution — rate source and observation timestamp — is stated once beneath the primary amount and governs every peso figure on the surface (UX-DR10).

**Given** the API returns no rate because the cached rate is older than the one-hour freshness window
**When** the surface renders
**Then** no peso figure appears anywhere on it, and nothing takes its place — no placeholder, no "rate unavailable", no dash
**And** the dollar figures render unchanged, because they were never dependent on the rate (AD-16, UX-DR10).

**Given** the FX disclosure
**When** it renders
**Then** it states explicitly that funds are held in a USD-pegged asset and not in pesos and that the peso value may move over the Cycle
**And** it is set at full ink in a disclosure block, never greyed, collapsed, truncated, or hidden behind "read more" (FR-22, UX-DR14).

**Given** the FX acknowledgement is unchecked
**When** the surface renders
**Then** the commit control is disabled with the blocker stated in text beneath it, never as a tooltip (FR-22, UX-DR14).

**Given** the Member checks the FX acknowledgement
**When** they later join
**Then** the acknowledgement is recorded (FR-22).

**Given** the terms surface
**When** the removal and irreversibility consequences are disclosed
**Then** it states before any commitment that an exhausted Stake removes them from the Room and forfeits an unreceived Payout, and that a Default is permanent on their record and cannot be erased (FR-16 disclosure, NFR-8, UX-DR13).

**Given** a terms row with a mechanic behind it
**When** the Member taps it
**Then** a plain-language sheet opens, stacking one level deep only, and no value is ever truncated — long values wrap (UX-DR6).

### Story 2.9: Funding check and single-signature commit

As a Member ready to join,
I want to know exactly what I am short before I try, and to commit in one signature,
So that I never discover a shortfall mid-transaction and never sign twice for one decision.

**Acceptance Criteria:**

**Given** an acknowledged terms screen
**When** the Funding check surface renders
**Then** it shows the required total broken into Stake and first Contribution, against the Member's balance of the Room asset (FR-21).

**Given** a balance below the required total
**When** the Funding check renders
**Then** it states the exact shortfall in dollars, the primary action is disabled, and the blocker is stated in text (FR-21, UX-DR14, UX-DR16).

**Given** a sufficient balance and a checked FX acknowledgement
**When** the Commit surface renders
**Then** it follows the commit pattern in order: what is given up as an amount, what is received and when as an amount and a date, what happens if a later Contribution cannot be paid, `commit-gap`, then one primary action (UX-DR5, UX-DR12).

**Given** the primary action
**When** it renders
**Then** it is labeled with the act and the amount — never "Confirm" and never "Continue" (UX-DR12).

**Given** the Member taps the primary action
**When** the flow proceeds
**Then** no confirmation dialog appears, no undo affordance is offered, and one signed transaction locks Stake and first Contribution together (UX-DR13, Story 2.5).

**Given** the signing handoff to the wallet
**When** it occurs
**Then** the app states what will be signed and its amount before leaving, and resolves to succeeded, rejected, or failed on return (UX-DR18).

**Given** the transaction fails or is rejected
**When** the app resolves it
**Then** it states the plain cause where knowable and states unambiguously whether money moved (UX-DR17).

**Given** the join reverts because the Registry Trust Score changed between display and submission
**When** the app surfaces the failure
**Then** it states plainly that the terms changed, shows the new score and the Stake it produces, and requires a fresh acknowledgement before retrying (AR-5).

### Story 2.10: Room filling and cancelled states

As a Member who has committed,
I want to see how close the room is to starting and what happens if it never does,
So that the wait is a known state rather than silence.

**Acceptance Criteria:**

**Given** a committed Member in a Room still admitting
**When** the Room filling surface renders
**Then** it shows Members joined out of Members needed and the days remaining in the open window (FR-6, UX-DR16).

**Given** the Room filling surface
**When** the cancellation terms render
**Then** they state that everything is returned in full if the Room does not fill, as a guarantee rather than a risk warning (FR-6, UX-DR20).

**Given** the open window deadline
**When** it is displayed
**Then** it is stated as an absolute date and time, not as a relative countdown alone (UX-DR25).

**Given** a Room that was cancelled
**When** the terminal state renders
**Then** it states what was returned, to which address, with the transaction link, and without apology framing — the guarantee worked (UX-DR16).

**Given** the Room fills and starts
**When** the Member next opens the app
**Then** they see the started Room with all Rounds, all Payout Positions and dates, including their own now-frozen position (FR-6, FR-11, FR-23).

## Epic 3: A cycle that pays out on schedule — and covers a miss without anyone losing money

The Room runs. Members contribute, see each other's status, and watch the schedule advance. Each Round's recipient receives the full Pot on the scheduled date — including when someone misses, through Grace Window, Slash, and Backstop. The Room closes and returns Stakes.

> **Cross-epic note.** AD-7 forbids the trust service from writing to Registry during an active Cycle. This epic's contracts therefore *emit* the events that carry score consequences — on-time contribution, cured contribution, Default, removal, Cycle completion — and Epic 4's trust service computes and commits the resulting score between Rooms. UI copy in this epic states the score consequence to the Member because they need it before a Grace Window closes; the Registry write itself belongs to Epic 4.

### Story 3.1: Per-Round Contribution escrow with Underwriter fee

As a Member,
I want to pay my contribution for the current round into the contract,
So that my money is held by code until it is released as someone's pot, with no person able to touch it.

**Acceptance Criteria:**

**Given** a started Room with an open Round
**When** a Member pays their Contribution
**Then** the amount is escrowed in the Room and the Member is recorded as paid for that Round (FR-8).

**Given** a Member who has already paid for the current Round
**When** they attempt to pay again
**Then** the invocation reverts (FR-8).

**Given** a Contribution submitted for a Round that is not the current open Round
**When** it is invoked
**Then** the invocation reverts (FR-8).

**Given** escrowed Contribution funds
**When** any code path is examined
**Then** the only paths that move them are Payout, Slash, and cancellation refund — no other entry point can transfer them, and no address holds authority to do so (FR-8, AR-6, NFR-3).

**Given** a Contribution is paid
**When** the Underwriter fee is applied
**Then** it is charged on the Contribution and never on the Payout, so the Pot is never reduced (FR-20).

**Given** any Contribution, fee, or Payout
**When** every code path that moves value is enumerated
**Then** no platform or operator address receives any portion of any of them — the platform takes no share of the fee and no share of the Pot (FR-20, PRD §11).

**Given** a Member whose KYC Attestation expires while the Cycle is in progress
**When** they pay a Contribution or their Payout Round arrives
**Then** the Contribution is accepted and the Payout executes normally — expiry blocks joining a *new* Room but never removes a Member from a Room already in progress, and never alters an existing obligation or entitlement (FR-1).

**Given** any Contribution
**When** it is stored
**Then** the amount is an integer stroop value and no floating point is used (AR-17).

**Given** a Contribution paid before its Round deadline
**When** it settles
**Then** the Room emits an on-time contribution event carrying the address, Round, and amount (AR-21).

**Given** a paid Contribution
**When** state is written
**Then** it uses `persistent` storage and the TTL of every entry touched is extended (AR-3).

### Story 3.2: Permissionless Round advance with full-Pot payout

As a Member scheduled to be paid,
I want the round to advance and the full pot to reach me on the scheduled date,
So that the money arrives without anyone deciding to send it.

**Acceptance Criteria:**

**Given** a Round whose deadline has passed and whose Contributions are all paid
**When** `advance_round` is invoked
**Then** the Member holding that Round's Payout Position receives the full Pot — Member count × Contribution amount — with no platform deduction (FR-12).

**Given** any address at all — a Member, the Underwriter, a keeper, or a stranger
**When** it invokes `advance_round` after the deadline
**Then** the invocation succeeds and its effect is identical regardless of caller (AR-13).

**Given** `advance_round` invoked before the Round deadline
**When** it is evaluated
**Then** it reverts (AR-13, AR-4).

**Given** `advance_round` invoked twice for the same Round
**When** the second call is evaluated
**Then** it is idempotent and produces no second Payout (AR-13).

**Given** the Round deadline
**When** it is evaluated
**Then** the comparison is the current ledger timestamp against a deadline stored in Room state, and no branch depends on whether any entry has expired (AR-4, AR-18).

**Given** a completed Cycle
**When** Payout history is examined
**Then** each Member has received exactly one Payout (FR-12).

**Given** a Payout
**When** it settles
**Then** the Room emits a payout event carrying the Round, recipient, and amount (AR-21).

**Given** the entire off-chain stack is unavailable
**When** a Member invokes `advance_round` directly against the contract
**Then** the Round advances and the Payout completes (AR-7, NFR-4).

### Story 3.3: Grace Window opens on a missed deadline and accepts a cure

As a member whose salary landed late,
I want 48 hours to pay before anything is taken from me,
So that being late is not treated the same as walking away.

**Acceptance Criteria:**

**Given** a Round deadline passes with one or more Contributions unpaid
**When** the transition is invoked
**Then** a Grace Window opens with a fixed 48-hour deadline stored in Room state as a ledger-timestamp offset (FR-14, AR-18).

**Given** an open Grace Window
**When** the Room state is evaluated
**Then** the Round does not advance and no Payout occurs while it remains open (FR-14).

**Given** a Member paying their Contribution within the open Grace Window
**When** it settles
**Then** the Contribution is accepted in full and recorded as cured
**And** the Room emits a cured-contribution event rather than a Default event (FR-14).

**Given** every outstanding Contribution has been cured
**When** the Grace Window resolves
**Then** the Round advances and the recipient is paid the full Pot on the same terms as an on-time Round (FR-14, FR-12).

**Given** an open Grace Window
**When** a notification is due
**Then** the Room state exposes Grace-Window-open and 24-hours-remaining conditions so the client can surface both required notifications (FR-14).

**Given** an open Grace Window
**When** other Members read Room state
**Then** it reads as awaiting contribution with the time remaining (FR-14, FR-9).

**Given** a Grace Window opening or resolving
**When** it settles
**Then** the Room emits an event carrying the Round, the affected addresses, and the window deadline (AR-21).

### Story 3.4: Atomic Slash, Backstop draw, and Payout on Default

As a Member scheduled to be paid in a round where someone defaulted,
I want the shortfall covered and my full pot paid on time regardless,
So that another member's default never costs me anything.

**Acceptance Criteria:**

**Given** a Grace Window that closes with a Contribution still unpaid
**When** the resolution is invoked
**Then** exactly the missed Contribution amount — not the whole Stake — is slashed from the defaulting Member's Stake (FR-15).

**Given** a defaulting Member whose remaining Stake is less than the missed Contribution
**When** the waterfall executes
**Then** the entire remaining Stake is slashed and the Backstop covers the difference (FR-15).

**Given** any branch of the waterfall
**When** the Round resolves
**Then** the scheduled recipient receives the full Pot in that same Round, with no delay and no reduction (FR-15, FR-12, SM-3).

**Given** Slash, Backstop draw, and Payout for a Round
**When** they execute
**Then** they occur within a single contract invocation that either fully succeeds or fully reverts
**And** no state exists in which a Slash succeeded but the recipient is underpaid (AR-9).

**Given** a resolved Default
**When** the waterfall completes
**Then** the Round advances immediately (FR-15).

**Given** a Default resolution
**When** it settles
**Then** the Room emits a Default event carrying the address, Round, amount slashed, and amount drawn from Backstop (AR-21).

**Given** the same chain history
**When** any independent observer recomputes the Slash and Backstop amounts
**Then** they arrive at identical figures, with no randomness involved (NFR-2, AR-11).

**Given** a resolved Slash
**When** any address — operator, Underwriter, or Member — attempts to reverse it
**Then** no entry point exists that permits reversal (NFR-3, AR-6).

### Story 3.5: Stake exhaustion removes a Member and the Backstop carries them

As an Underwriter,
I want a member whose stake is gone removed from the room with the backstop covering their remaining rounds,
So that the cycle completes for everyone else exactly as scheduled.

**Acceptance Criteria:**

**Given** a Slash that leaves a Member's Stake at zero
**When** it completes
**Then** the Member is removed from the Room at that moment (FR-16).

**Given** a removed Member
**When** the remaining Rounds of the Cycle execute
**Then** the Backstop covers that Member's Contribution for every remaining Round (FR-16).

**Given** a removed Member who has not yet received their Payout
**When** their scheduled Round arrives
**Then** they do not receive a Payout and the Round advances for everyone else (FR-16).

**Given** a removed Member who already received their Payout
**When** removal completes
**Then** they retain it and the Backstop absorbs the loss (FR-16).

**Given** a removal
**When** it settles
**Then** the Room emits an abandonment event carrying the address and the Round at which removal occurred (AR-21).

**Given** a removal
**When** Room membership is examined
**Then** removal through Stake exhaustion is the only mechanism by which membership changes after Room start (FR-6, FR-16).

### Story 3.6: Room close returns Stakes and settles the Underwriter

As a Member who completed every round,
I want my stake back and the room closed cleanly,
So that the only money I am left without is the money I chose to contribute.

**Acceptance Criteria:**

**Given** a Room whose final Round has completed
**When** close is invoked
**Then** every Member who completed all Rounds has their full remaining Stake returned to their address (FR-7).

**Given** Room close
**When** the Underwriter is settled
**Then** undrawn Backstop capital returns to the Underwriter and accrued fees are settled (FR-7, FR-20, AR-14).

**Given** Room close
**When** it settles
**Then** the Room emits a Cycle-completion event per Member, carrying whether they completed the Cycle with zero Defaults, for the trust service to consume (FR-7, AR-21).

**Given** a closed Room
**When** any further Contribution, Payout, or membership change is attempted
**Then** it reverts.

**Given** close is invoked before the final Round has completed
**When** it is evaluated
**Then** it reverts (AR-4).

**Given** a closed Room
**When** its remaining balance is examined
**Then** no Member or Underwriter value remains held by the contract.

### Story 3.7: Event-sourced read models and query API

As a Member,
I want the app to show me the whole cycle at once without asking me to read a ledger,
So that state I can act on is legible while the chain remains the thing that decides.

**Acceptance Criteria:**

**Given** the indexer
**When** it builds read models
**Then** it is built only from contract events, never from state polling (AR-21).

**Given** the read model store
**When** it is dropped and rebuilt by replaying chain events from genesis of the Room
**Then** the resulting models are identical (AR-1, AR-24).

**Given** the API
**When** any request is served
**Then** it is read-only and exposes no path that submits a transaction or adjudicates an outcome (AR-7).

**Given** a value that determines money movement or Member obligation
**When** the client needs it at a decision point
**Then** it is read from chain state, not from the read model (AR-1).

**Given** the indexer or API is unavailable
**When** a Member invokes a Room entry point directly
**Then** every in-flight Room remains fully operable (AR-7, NFR-4).

**Given** read models
**When** they are stored
**Then** they live in PostgreSQL as rebuildable caches and are never treated as authoritative (AR-24).

**Given** the rate adapter has a cached USD/PHP rate
**When** a client requests it through the API
**Then** the response carries the rate, its source identifier, and its observation timestamp together — never a rate without both (AD-16).

**Given** the cached rate is older than the one-hour freshness window
**When** a client requests it
**Then** the API omits the rate entirely rather than serving it stale, so the freshness decision is made in exactly one layer and no client can disagree with another (AD-16).

**Given** the rate adapter or its upstream provider is unavailable
**When** any Member joins, contributes, or receives a Payout
**Then** none of those paths is blocked — rate unavailability degrades presentation only (AD-16, AR-7).

### Story 3.8: Room home — current round, my obligation, my payout date

As a Member,
I want one screen that tells me what I owe, when, and when I get paid,
So that the app never leaves me wondering where I stand.

**Acceptance Criteria:**

**Given** a started Room and a connected Member
**When** Room home renders
**Then** it shows the current Round, the Member's obligation, their Payout date, and the schedule (FR-23, UX-DR8).

**Given** the schedule
**When** it renders
**Then** it shows all Rounds, all Payout Positions and dates, and which Rounds are complete (FR-23).

**Given** a schedule row
**When** it renders
**Then** the current Round carries a 3px accent marker, completed Rounds carry a paid pill, the Member's own Round is set in `body-strong`, and rows never reorder (UX-DR6).

**Given** the Round open with the Member's Contribution unpaid
**When** Room home renders
**Then** the primary action is the payment, labeled with act and amount (UX-DR12, UX-DR19).

**Given** the Round open with the Member's Contribution paid
**When** Room home renders
**Then** it confirms with the date and states the next obligation and its date immediately (UX-DR16).

**Given** any Round deadline shown
**When** it renders
**Then** it is stated as an absolute date and time, never as a relative countdown alone (UX-DR25).

**Given** Room home
**When** the Member pulls to refresh
**Then** chain state is re-read; pull-to-refresh exists on Room home and Underwriter room only (UX-DR19).

**Given** the app is offline or a wallet session has expired
**When** Room home renders
**Then** last-known state is shown with an "as of" timestamp and the schedule remains readable (UX-DR16, UX-DR17).

### Story 3.9: Contribute

As a Member,
I want to pay this round's contribution in a few taps,
So that meeting my obligation is the easiest thing the app asks of me.

**Acceptance Criteria:**

**Given** an open Round with the Member unpaid
**When** the Contribute surface renders
**Then** it states the amount due and the deadline, with the dollar figure as the number of record and any peso line beneath it prefixed `≈`, attributed once on the surface with its rate source and observation timestamp
**And** when the API returns no rate because the cached one is outside the one-hour window, no peso figure appears and nothing replaces it (FR-8, AD-16, UX-DR10).

**Given** the Contribute surface
**When** the amount is labeled
**Then** it is named a *contribution* and is never confusable with stake, pot, or fee (UX-DR11).

**Given** the Member commits
**When** the flow proceeds
**Then** it follows the commit pattern with one primary action labeled with act and amount, no confirmation dialog, and no undo affordance (UX-DR12, UX-DR13).

**Given** the signing handoff
**When** it occurs
**Then** the app states what will be signed and its amount before leaving and resolves to a definite state on return (UX-DR18).

**Given** the transaction is submitted but unconfirmed
**When** the Member navigates away and returns
**Then** pending state is reconstructed from chain state with the action named, and the app is never blocked as a whole (UX-DR16, UX-DR18).

**Given** the transaction fails
**When** the app resolves it
**Then** it states the plain cause and states unambiguously whether money moved (UX-DR17).

### Story 3.10: Members status list

As a Member,
I want to see who has paid this round,
So that I know the state of the room without having to ask anyone.

**Acceptance Criteria:**

**Given** a started Room
**When** the Members surface renders
**Then** every Member's status for the current Round is shown as paid, pending, in grace, or defaulted (FR-9).

**Given** any Member row
**When** it renders
**Then** it shows the handle and a status pill only — no avatar, name, or photo (FR-9, UX-DR6, NFR-7)
**And** the handle comes from the shared derivation module, so it matches this Member's handle on every other surface and in every API response (AD-17, UX-DR26).

**Given** a status
**When** it renders
**Then** the text carries the meaning and color only reinforces it; no state is encoded in color alone (UX-DR21).

**Given** a status pill
**When** it renders
**Then** it is outline-only and colored from the money-state palette, and no other status values exist beyond the four (UX-DR2, UX-DR6).

**Given** a Member in a Grace Window
**When** other Members view the list
**Then** the handle is shown with the state and time remaining, and no blame or judgement appears in the copy (FR-14, UX-DR20).

### Story 3.11: Grace banner and the highest-stakes state in the product

As a Member who missed a deadline,
I want an unmissable warning that states exactly what happens in 47 hours,
So that the cost of not paying is never a surprise.

**Acceptance Criteria:**

**Given** the connected Member has an open Grace Window
**When** any surface renders
**Then** a persistent grace banner appears over the grace wash with a hairline border in the grace state color, and it survives navigation across every surface, not only Room home (FR-14, UX-DR25).

**Given** the grace banner
**When** it renders
**Then** it states the hours remaining, the amount due, and the exact consequence at expiry — the amount that will be slashed from their Stake and the 150-point score penalty (FR-14, FR-15, UX-DR20).

**Given** the grace banner
**When** the Member attempts to dismiss it
**Then** no dismissal affordance exists; it clears only on a state change (UX-DR6).

**Given** the grace banner
**When** a screen reader encounters it
**Then** it is an `aria-live="assertive"` region and state changes on Room home announce (UX-DR22).

**Given** the grace banner and the FX disclosure
**When** contrast is measured
**Then** both meet WCAG AAA for body text (UX-DR22).

**Given** an open Grace Window belonging to another Member
**When** the connected Member's Room home renders
**Then** it reads as awaiting contribution with time remaining, in the blame-free variant (FR-14, UX-DR20).

**Given** V1's in-app-only notification constraint
**When** the Grace Window opens and again at 24 hours remaining
**Then** both required notifications are delivered as Room home state and the persistent banner
**And** the known reach gap is recorded rather than implied to be solved (FR-14, UX-DR25).

### Story 3.12: Round detail, default resolved, and removal

As a Member,
I want to see what actually happened in a round where someone missed,
So that I can see the system working rather than guess at it.

**Acceptance Criteria:**

**Given** a Member tapping a schedule row
**When** Round detail renders
**Then** it shows that Round's outcome: who was paid, what it cost, and what resolved (FR-12, FR-15, UX-DR8).

**Given** a Round in which a Default was resolved
**When** Round detail renders
**Then** it states plainly who missed, what covered it, and that the recipient was paid in full and on time — presented as the system working (FR-15, UX-DR16, UX-DR20).

**Given** a Payout the connected Member received
**When** it renders
**Then** it shows the full Pot with no deduction, the date, and a transaction link, with no celebration, confetti, or animation (FR-12, UX-DR11, UX-DR19).

**Given** the connected Member was removed through Stake exhaustion
**When** the terminal state renders
**Then** it states completely: the remaining Rounds now covered by the Backstop, the Payout forfeited if not yet received, and the score penalty applied
**And** the copy offers no path back, because none exists in V1 (FR-16, UX-DR16, UX-DR20).

**Given** any Default entry shown to the Member it belongs to
**When** it renders
**Then** it states that this stays on their record (NFR-8, UX-DR15).

### Story 3.13: Cycle summary at Room close

As a Member who finished a cycle,
I want a final record of what I put in, what I got back, and what it earned me,
So that the cycle ends with a statement rather than silence.

**Acceptance Criteria:**

**Given** a closed Room
**When** the Cycle summary renders
**Then** it shows Rounds completed, Stake returned, and the score change (FR-7, UX-DR8).

**Given** the Cycle summary
**When** the carry-forward is stated
**Then** it states concretely what this score means for the next Room's Stake, as a multiple (UX-DR15).

**Given** the Cycle summary
**When** amounts render
**Then** Stake, Contribution, Pot, and fee are each labeled with their own noun and never confusable (UX-DR11).

**Given** the Cycle summary
**When** copy is reviewed
**Then** it is a plain statement of fact with no congratulation the mechanics do not earn (UX-DR20).

### Story 3.14: Underwriter room — live and maximum exposure

As an Underwriter,
I want to see what I am exposed to right now and at worst,
So that I know my downside at every point in the cycle.

**Acceptance Criteria:**

**Given** a running Room
**When** the Underwriter room surface renders
**Then** it shows live exposure, maximum possible exposure, contributions received, and any open Grace Window (FR-17, UX-DR8).

**Given** a Slash or Backstop draw
**When** it settles
**Then** live exposure updates to reflect it (FR-17).

**Given** the Underwriter room on a wide viewport
**When** it renders
**Then** it shows more rows and wider tables while remaining a single column, and nothing about it is desktop-only (UX-DR24).

**Given** the Underwriter room
**When** the Underwriter pulls to refresh
**Then** chain state is re-read (UX-DR19).

**Given** exposure figures
**When** they render
**Then** they use the amount roles with `tabular-nums` and the dollar figure as the number of record (UX-DR3, UX-DR10).

## Epic 4: Trust that compounds

A Member's score moves on what they actually did, they can recite why it moved, and the score they end a Cycle with visibly changes what their next Room costs them.

### Story 4.1: Deterministic, versioned Trust Score computation

As Pal3,
I want the trust service to compute a Member's score as a pure function of chain event history,
So that the same events always yield the same score and no one can argue with the result.

**Acceptance Criteria:**

**Given** a Member's chain event history
**When** the trust service computes their score
**Then** it applies: +10 on-time Contribution, +2 Contribution cured within the Grace Window, −150 Default, +50 Cycle completed with zero Defaults, −400 Room abandoned through Stake exhaustion (FR-10).

**Given** any computed score
**When** it is produced
**Then** it is clamped to the range 0–1000 after every update (FR-10).

**Given** a Member who has completed zero Cycles
**When** their score is computed
**Then** it does not exceed 400 (FR-2).

**Given** the same sequence of events
**When** the score is computed any number of times, on any machine
**Then** the result is identical, with no randomness, wall-clock time, or off-chain state as an input (FR-10, NFR-2, AR-11).

**Given** the scoring function
**When** it is released
**Then** it carries an explicit version identifier (AR-22).

**Given** the trust service
**When** it reads its inputs
**Then** it reads only contract events, never a read model or any other off-chain store, as the basis for a score (AR-1, AR-22).

**Given** an event sequence containing a Default
**When** the score is computed
**Then** the Default's effect is permanent in the history and is never removed by a later event (NFR-8).

**Given** the trust service implementation
**When** its dependencies are examined
**Then** the scoring logic is chain-agnostic service code, with chain-specific event decoding isolated at its boundary, so the mechanism survives a later migration toward a peso rail (NFR-11).

**Given** the trust service is unavailable
**When** any Room operation occurs
**Then** no Payout, Contribution, or Round advance is blocked by its absence (AR-7, NFR-4).

### Story 4.2: Registry commit between Rooms

As a Member,
I want the score I earned in one room to be written to the registry where my next room will read it,
So that reliability in one circle is worth something in the next.

**Acceptance Criteria:**

**Given** a computed score and no active Cycle involving that Member
**When** the trust service commits
**Then** it writes the score to Registry and records the scoring function version alongside the commit (AR-22, FR-10).

**Given** a Member in an active Cycle
**When** the trust service has a newer computed score for them
**Then** it does not write to Registry until the Cycle has closed (AR-7).

**Given** a Registry Trust Score write
**When** it settles
**Then** the Registry emits an event carrying the address, previous value, and new value (AR-21).

**Given** the trust service
**When** it operates
**Then** it writes only to Registry and never to a Room contract, and never submits an ordering (AR-5, AR-7).

**Given** a Room in progress
**When** a Member's Registry score changes after the Room started
**Then** the Room's snapshotted score and the Stake derived from it are unaffected (AR-5, FR-13).

**Given** a committed score
**When** an observer recomputes it from chain events using the recorded function version
**Then** they arrive at the same value (NFR-2).

### Story 4.3: Trust surface — score, consequence, and every event that moved it

As a Member,
I want to see my score, what it costs or saves me, and every event that changed it,
So that I can say out loud what my score is and why.

**Acceptance Criteria:**

**Given** a verified Member
**When** the Trust surface renders
**Then** it shows the score, the fill track at score/1000, and the derived consequence — never fewer than all three (FR-23, UX-DR6, UX-DR15).

**Given** the consequence line
**When** it renders
**Then** it states the Stake the score produces as a multiple of one Contribution (FR-13, UX-DR15).

**Given** the score history
**When** it renders
**Then** each event is a plain sentence with its delta and date — for example "Paid round 3 on time · +10 · 24 September" — with no icons, categories, or aggregation (FR-23, UX-DR15).

**Given** a Default entry in the history
**When** it renders
**Then** it states that this stays on their record (NFR-8, UX-DR15).

**Given** a newly verified Member whose score history is empty — the state every Member is in at pilot start
**When** the Trust surface renders
**Then** the score and its consequence render normally, and the history area states that nothing has moved the score yet and names the first event that will: their next on-time Contribution
**And** it shows no illustration, no encouragement, and no placeholder rows (UX-DR27).

**Given** the Trust surface
**When** it renders
**Then** the score appears as a number with its consequence and never as a badge, level, tier, streak, or grade (UX-DR7, UX-DR15).

**Given** any surface other than Trust and the Room terms screen
**When** it renders
**Then** the score does not appear, and never as a decoration beside a handle (UX-DR15).

**Given** a Member viewing any surface
**When** other Members are shown
**Then** their scores are not visible to Members; only Underwriters see them, on the Applicants surface (FR-5, FR-9, UX-DR15).

**Given** the score history
**When** it renders
**Then** no money-state or accent color is applied to any history row (UX-DR2).

### Story 4.4: Next threshold and cycle carry-forward

As a Member,
I want to know what my score is about to buy me,
So that the number connects to something I actually care about.

**Acceptance Criteria:**

**Given** a score below 1000
**When** the Trust surface renders the next threshold
**Then** it is stated in member terms — the points needed and the Stake multiple they unlock — rather than as a bare integer target (UX-DR15).

**Given** a Member who has just completed a Cycle
**When** the Cycle summary renders
**Then** it states the score change and what the resulting score means for the next Room's Stake, as a multiple (FR-7, FR-10, UX-DR15).

**Given** the carry-forward statement
**When** it renders
**Then** it names the concrete change — for example that the next Room's Stake is 1.54× instead of 1.6× — rather than a general encouragement (UX-DR20).

**Given** any copy on these surfaces
**When** it is reviewed
**Then** it is sentence case, in full sentences, with no exclamation marks and no congratulation the mechanics do not earn (UX-DR20).

### Story 4.5: Independent score reproduction

As a Member or a reviewer,
I want to be able to recompute any score from public chain history myself,
So that "the score is reproducible by any observer" is something I can check rather than something I am told.

**Acceptance Criteria:**

**Given** a Member address and a scoring function version
**When** the reproduction tool is run against public chain history
**Then** it outputs the score and the full event sequence that produced it (NFR-2, FR-10).

**Given** the tool's output and the value committed in Registry
**When** they are compared
**Then** they match (NFR-2, AR-22).

**Given** the tool
**When** it is published
**Then** it runs against public chain data with no access to any Pal3 off-chain store (AR-1).

**Given** a scoring function version change
**When** the tool is run with the prior version
**Then** it reproduces scores as they were computed under that version (AR-22).

## Epic 5: Pilot readiness — testnet to mainnet

The contracts are public, audit-ready, and deployed along a path where no Room holds real value until the audit-readiness review has passed.

### Story 5.1: Contracts published under Apache 2.0 from first deployment

As a Member being asked to escrow money in code,
I want the contract source public from the first deployment,
So that I am not asked to trust something I cannot inspect.

**Acceptance Criteria:**

**Given** the first deployment of any contract to any network
**When** it occurs
**Then** the corresponding source is already public under Apache 2.0 (NFR-9, AR-12).

**Given** the published repository
**When** it is inspected
**Then** every crate under `contracts/` carries an Apache 2.0 header and the repository carries the licence file, including the explicit patent grant that motivated the choice over MIT (AR-12).

**Given** a deployed contract WASM
**When** a third party rebuilds from the published source at the tagged commit
**Then** they can verify the deployed WASM corresponds to that source.

**Given** the published repository
**When** it is inspected for secrets
**Then** no operator key, provider credential, or environment secret is present in source or history (AR-23).

### Story 5.2: Testnet deployment with recorded addresses

As a developer on Pal3,
I want the full system deployed to testnet with addresses configured through environment,
So that the pilot is exercised end to end before any real value is involved.

**Acceptance Criteria:**

**Given** the built contracts
**When** they are deployed to Stellar testnet
**Then** Registry, Factory, and a Room deployed by the Factory are all operating, and their addresses are recorded (AR-25, AR-2).

**Given** the deployed addresses and network passphrase
**When** services and the client are built
**Then** they are supplied through environment configuration, with no literal in source (AR-23).

**Given** a testnet deployment
**When** a full Cycle is exercised
**Then** a Room forms, runs every Round, resolves at least one induced Default through the waterfall, and closes — with the recipient paid in full in every Round (SM-1, SM-2, SM-3).

**Given** generated bindings
**When** the client is built against the deployed contracts
**Then** the bindings match the deployed contract interfaces and were produced by the build rather than by hand (AR-8).

### Story 5.3: Invariant verification against the three load-bearing claims

As a reviewer or a Member,
I want the invariants the product's promise rests on verified as properties, not asserted in prose,
So that "no one can take your money and nothing is random" is demonstrated.

**Acceptance Criteria:**

**Given** the test harness established in Story 1.2
**When** these invariants are verified
**Then** each is expressed as an automated test in that harness and runs in CI, rather than as a one-time manual review (NFR-1).

**Given** the full contract surface
**When** every entry point is enumerated and tested
**Then** no address — operator, Underwriter, admin, or Member — can transfer escrow, alter a Payout Position after Room start, or reverse a Slash (NFR-3, AR-6).

**Given** the Registry admin authority
**When** its reachable effects are enumerated
**Then** they are limited to issuing and revoking Attestations and writing Trust Scores, and cannot touch any Room's funds (AR-6).

**Given** the Round resolution path
**When** a Slash, Backstop draw, and Payout are forced to fail partway
**Then** the entire invocation reverts and no state exists in which the recipient is underpaid (AR-9).

**Given** the contract source
**When** it is searched for randomness or non-deterministic inputs
**Then** none is present, and ordering, Stake sizing, Slash amounts, and score inputs are pure functions of committed state and ledger timestamp (NFR-2, AR-11).

**Given** any time-based transition
**When** the code path is inspected
**Then** no branch depends on whether a storage entry has expired (AR-4).

**Given** a Room with the off-chain stack fully unavailable
**When** a Member invokes contribution and Round advance directly
**Then** both succeed (AR-7, NFR-4).

**Given** an upgrade mechanism, if one is present at all
**When** its reach is examined
**Then** it cannot alter an in-flight Room (AR-6).

### Story 5.4: Transaction cost measured against a real contribution

As a Member contributing about ₱1,000 a week,
I want the cost of using Pal3 to be negligible against what I pay in,
So that the product is actually cheaper than the free alternative it replaces.

**Acceptance Criteria:**

**Given** a testnet Cycle exercised end to end
**When** transaction costs are measured
**Then** the per-Round cost to a Member is recorded for join, contribute, and Round advance (NFR-5).

**Given** the measured per-Round cost
**When** it is compared against a ₱1,000-equivalent Contribution
**Then** it is negligible by the standard the PRD sets, and the figure is recorded rather than estimated (NFR-5).

**Given** a measured cost that is not negligible
**When** the result is reviewed
**Then** it is raised as a product-level blocker before mainnet, not absorbed as an engineering detail (NFR-5).

### Story 5.5: Backstop adequacy verified against a full exposure table

As the pilot Underwriter,
I want the backstop formula checked against every exposure case including the worst one,
So that I am not posting capital against a formula that has never been tested.

**Acceptance Criteria:**

**Given** the FR-19 formula of 2 × Contribution × Member count
**When** a full exposure table is produced across Member counts 5 through 10
**Then** every Default scenario is enumerated, including the Round-1 recipient defaulting immediately after Payout with roughly nine Contributions outstanding against a Stake covering roughly two (PRD §13 item 2, §14).

**Given** the completed exposure table
**When** the formula is evaluated against the worst case
**Then** either the formula is confirmed adequate, or a revised formula is specified and Story 2.2's acceptance criteria are updated to match (FR-19).

**Given** the exposure table
**When** it is complete
**Then** it is recorded as a planning artifact before Backstop capital is posted for the pilot (PRD §13 item 2).

### Story 5.6: Pre-mainnet open-question closure

As the founder,
I want the PRD's two build-affecting open questions closed before recruitment,
So that a known gap is a decision on the record rather than something a member discovers.

**Acceptance Criteria:**

**Given** PRD §13 item 1 — Grace Window abuse, where a Member could cure every Round indefinitely, taking +2 rather than −150 and delaying every Payout by up to 48 hours
**When** the decision is made
**Then** it is recorded, and if an escalation rule is adopted (for example, treating the next cure as a Default after N cures in one Cycle) Story 3.3's acceptance criteria are updated to specify it before implementation (PRD §13 item 1).

**Given** the in-app-only notification gap recorded in `EXPERIENCE.md § Notification & Reach`
**When** the mainnet decision is taken
**Then** either SMS escalation at Grace-Window open and at 24 hours remaining is scoped as a story, or the gap is accepted in writing with the reason (UX-DR25).

**Given** PRD §13 item 4 — the named pilot cohort
**When** recruitment begins
**Then** the cohort of 5–10 is identified, and each member has consented to a pilot with no dispute process, no admin override, and no way to reverse a Slash (PRD §5, §13 item 4).

**Given** any open question closed here
**When** the decision is recorded
**Then** the affected story's acceptance criteria are updated in this document, so a dev agent never implements against a superseded specification.

### Story 5.7: Audit-readiness review gates the mainnet pilot

As a Member putting real money into the pilot,
I want the contracts reviewed for audit-readiness before any room holds real value,
So that the first time this code holds my savings is not the first time anyone has scrutinized it.

**Acceptance Criteria:**

**Given** contracts holding escrow, Stake, and Backstop
**When** the audit-readiness review is conducted
**Then** it covers every value-bearing path and its findings are recorded (NFR-1, brief milestone T2).

**Given** an unresolved finding that affects a value-bearing path
**When** mainnet deployment is considered
**Then** it does not proceed (NFR-1).

**Given** a passed audit-readiness review, a verified exposure table, and closed open questions
**When** mainnet deployment proceeds
**Then** Registry, Factory, and the pilot Room are deployed to mainnet and their addresses recorded (AR-25).

**Given** any mainnet Room
**When** it opens with real value
**Then** the audit-readiness review has already passed — this is an absolute precondition with no exception path (NFR-1).
