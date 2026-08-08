---
title: "Pal3 — on-chain stablecoin savings circle"
status: final
created: 2026-08-08
updated: 2026-08-08
---

# PRD: Pal3

## 0. Document Purpose

This PRD specifies the V1 pilot of Pal3 for two readers: the founder building it, and Stellar Community Fund reviewers assessing whether it can be built. It defines capabilities, not implementation — chain-level and contract-level decisions live in `addendum.md` and downstream architecture work.

It builds on the completed product brief at `_bmad-output/planning-artifacts/briefs/brief-paluwagan3-2026-08-08/` and does not restate it. Vocabulary is Glossary-anchored: §3 defines every domain noun, and the rest of the document uses those terms verbatim. Features are grouped in §4 with functional requirements nested and numbered globally (FR-1…FR-N) so downstream artifacts keep stable references. Inferred decisions carry inline `[ASSUMPTION]` tags and are indexed in §9.

**Scope boundary:** this document specifies the V1 pilot — one room, 5–10 members, one underwriter, run on Stellar/Soroban with a USD-pegged stablecoin. Post-MVP capability is named only where it constrains a V1 decision.

## 1. Vision

Pal3 is the Filipino paluwagan with the two failure modes engineered out. A group saves together, each member takes the whole pot in turn — but the smart contract holds the money instead of a person, so the organizer cannot run off with it, and a trust engine makes sure that when someone stops paying, everyone else still gets paid in full.

The traditional version works beautifully until it doesn't. One default cascades to everyone downstream. One dishonest organizer takes the pool. The Repa Paluwagan collapse cost roughly ₱2B across Davao and Bohol, and the SEC issues advisories against "Online Paluwagan" schemes on a rolling basis. The result is that a tool perfectly suited to people without access to bank credit only works among people who already trust each other completely — which caps it at the size of a friendship group and makes every participant one bad actor away from losing months of savings.

Pal3 removes the human organizer entirely and replaces social trust with a portable, identity-anchored **Trust Score** that follows a member across every **Room** they ever join. Members get lump-sum access without the fear. Reliable savers accumulate something they have never had before: a financial reputation that is theirs, provable, and worth something.

## 1a. Why Now

Timing is load-bearing here, in three ways:

- **The formal system is losing ground.** Philippine account ownership *fell* to 50% in 2025 from 56% in 2021, with 44% of adults unbanked on low income or missing documents. Informal savings is not a legacy behavior being displaced — it is absorbing people the formal system is shedding.
- **Regulated peso rails became real.** The BSP's sandbox approval of PHPC established that a peso stablecoin can exist inside Philippine regulation. Pal3's Phase 2 depends on that category existing; two years ago it did not.
- **The category's failures are now diagnosable.** On-chain ROSCAs (HaloFi, WeTrust) rotated correctly but were pseudonymous, so a defaulter's only cost was abandoning a wallet. That is a specific, fixable mechanism failure — and fixing it is the entire product thesis.

## 2. Target User

### 2.1 Jobs To Be Done

**Members (income earners):**
- *Functional:* reach a lump sum I cannot save alone and cannot borrow from a bank.
- *Functional:* save on a schedule I can actually keep, enforced by something other than my own willpower.
- *Emotional:* participate without the fear of losing months of contributions to a defaulter or an organizer.
- *Social:* save alongside people I know without having to police them, or damage the relationship when someone misses.
- *Contextual:* build a record of reliability that means something outside this one circle.

**Underwriters:**
- *Functional:* earn a return on idle capital by pricing and absorbing default risk.
- *Functional:* see enough about a prospective member's history to decide whether to accept them.
- *Emotional:* bound my downside — know the maximum I can lose before I commit.

### 2.2 Non-Users (V1)

- **The unbanked.** V1 requires employment records and bank details to anchor identity. This excludes the population in the long-term vision, deliberately and temporarily.
- **Anyone unwilling to hold a USD-pegged stablecoin.** V1 has no peso rail; members carry FX exposure for a full cycle and must consent to it.
- **Members outside the founder's recruitable network.** V1 is a closed pilot, not open enrolment.
- **Institutional or employer-sponsored groups.** Post-MVP.

### 2.3 Key User Journeys

- **UJ-1. Josel joins a paluwagan he can't be burned by.**
  > Josel, 29, a BPO team lead in Cebu, lost ₱18,000 two years ago when an officemate's paluwagan collapsed three rounds before his turn. He has not joined one since. A workmate invites him to a Pal3 **Room** of eight. Unauthenticated, on his phone, he installs the app and completes **KYC Verification** — ID, employment record, bank details — through the provider's flow. He sees his starting **Trust Score** and, derived from it, the **Stake** he must lock: a little under twice one contribution. The Room's terms are on one screen — ₱1,000-equivalent weekly, eight rounds, his **Payout Position** is 6th, and the **Underwriter** covers any shortfall beyond a defaulter's stake. He locks his stake and his first contribution. **Climax:** the Room fills and starts, and the schedule shows all eight payout dates including his own — a date he can plan around, from a contract that cannot decide otherwise. **Resolution:** he is committed for eight weeks and knows exactly what he is owed and when. **Edge case:** if the Room fails to fill within its open window, his stake and contribution are returned in full and the Room never starts.

- **UJ-2. Marivic opens a Room and picks who's in it.**
  > Marivic has idle capital and a subscription tier permitting one Room of up to ten members. She configures cadence, contribution amount, and size, then posts the **Backstop** capital the tier requires. As members request to join she sees each one's Trust Score, cycles completed, and default history — never their identity documents. She rejects one applicant with two prior defaults and accepts the rest. **Climax:** the Room reaches its member count and starts; her per-contribution fee begins accruing. **Resolution:** she monitors a dashboard showing exposure, contributions received, and any Room in a **Grace Window**.

- **UJ-3. Dennis misses a payment and nobody else loses money.**
  > Dennis's salary lands two days late in week four. He misses the contribution deadline. **Path:** the contract opens a 48-hour **Grace Window** and notifies him; the round does not advance and no one else is affected yet. He pays 20 hours later. His contribution is accepted, his Trust Score takes a small penalty rather than a slash, and the round advances on schedule. **Climax:** the scheduled recipient is paid in full, on time, and never knew there was a problem. **Edge case:** had Dennis not paid within 48 hours, the contract would slash exactly the missed amount from his Stake, top up any remainder from the Underwriter's Backstop, pay the recipient in full, and apply a **Default** penalty to his Trust Score. If his Stake were exhausted, he would be removed from the Room and the Backstop would carry his remaining rounds.

- **UJ-4. Josel gets paid, and gets something he can keep.**
  > Week six. The contract releases the full **Pot** to Josel — eight contributions, minus nothing. **Climax:** the money arrives in his wallet on the scheduled date without anyone deciding to send it. **Resolution:** he continues contributing for the remaining two rounds, and his Trust Score rises on cycle completion. When the Room closes he has a score that makes his next Room cheaper to enter — a smaller stake for the same contribution. That is the first time saving reliably has ever earned him anything portable.

## 3. Glossary

- **Room** — one paluwagan instance: a fixed set of Members, one Underwriter, a fixed Contribution amount and Cadence, running for exactly as many Rounds as it has Members. Rooms do not overlap membership requirements; a Member may belong to several Rooms.
- **Member** — a KYC-verified individual participating in a Room, who contributes each Round and receives the Pot exactly once per Cycle.
- **Underwriter** — the party who opens a Room, admits or rejects Members, posts Backstop capital, and earns a Fee per Contribution in exchange for covering shortfalls.
- **Cycle** — one complete run of a Room, from start to the final Payout. Length in Rounds equals the Member count.
- **Round** — one Cadence interval within a Cycle: every Member contributes once, one Member receives the Pot.
- **Cadence** — the fixed interval between Rounds (V1: weekly).
- **Contribution** — the fixed amount each Member pays each Round.
- **Pot** — the sum of all Contributions in a single Round, paid in full to one Member.
- **Payout** — the transfer of a Pot to its scheduled recipient.
- **Payout Position** — a Member's assigned Round for receiving the Pot, derived from Trust Score at Room start.
- **Trust Score** — a network-wide integer, 0–1000, held per Member across all Rooms. Determines Payout Position and Stake size.
- **Stake** — collateral a Member locks at join, sized inversely to Trust Score, slashable to cover that Member's missed Contributions.
- **Slash** — the transfer of value out of a Member's Stake to cover their own missed Contribution.
- **Backstop** — capital posted by the Underwriter, drawn on when a Slash does not fully cover a shortfall.
- **Grace Window** — a fixed 48-hour period after a missed Contribution deadline during which the Member may still pay without being recorded as a Default.
- **Default** — a Contribution not paid by the close of its Grace Window.
- **Fairness Floor** — the constraint preventing a Member from being assigned the final Payout Position in more than two consecutive Cycles within the same Room.
- **KYC Attestation** — an opaque verification record issued by the KYC Provider and held by Pal3, proving a Member passed verification without Pal3 holding their identity documents.
- **KYC Provider** — the licensed third party that performs identity verification and is the sole custodian of Member identity documents.

## 4. Features

### 4.1 Identity and Verification

**Description:** Every Member is a verified real person before they can join any Room. Verification runs entirely through the KYC Provider; Pal3 never receives or stores identity documents, employment records, or bank details. Pal3 holds only a KYC Attestation and an opaque provider reference. This is what makes a Default carry a real-world consequence and is the foundation the entire Trust Score rests on. Realizes UJ-1.

**Functional Requirements:**

#### FR-1: Member identity verification

A prospective Member can complete identity verification through the KYC Provider before joining any Room. Realizes UJ-1.

**Consequences (testable):**
- A wallet with no valid KYC Attestation cannot join a Room; the join transaction reverts.
- Pal3 systems store only: attestation status, verification tier, issue date, provider reference. No name, document image, employment record, or bank detail is persisted by Pal3.
- A Member with an expired attestation cannot join a *new* Room but is not removed from a Room already in progress.

#### FR-2: Verification tier sets starting Trust Score

The system assigns a Member's initial Trust Score from their verification tier. Realizes UJ-1.

**Consequences (testable):**
- Basic verification (government ID only) yields a starting Trust Score of 300. `[ASSUMPTION]`
- Full verification (ID + employment record + bank detail) yields 400. `[ASSUMPTION]`
- A Member cannot hold a Trust Score above 400 until they have completed at least one Cycle.

#### FR-3: One human, one identity

The system prevents a single verified individual from holding multiple Member accounts.

**Consequences (testable):**
- The KYC Provider's duplicate-detection result is enforced at attestation issue; a second attestation for the same individual is rejected.
- An attempt to join a Room with a wallet whose attestation is flagged as duplicate reverts.

**Feature-specific NFRs:**
- Identity document handling is out of Pal3's technical boundary by design; the KYC Provider is the sole personal information controller for document data.

### 4.2 Room Lifecycle

**Description:** An Underwriter creates a Room with fixed parameters, Members join until it fills, and it then runs a fixed number of Rounds and closes. Parameters are immutable once the Room starts — no one, including the Underwriter and the platform, can change contribution amounts, membership, or ordering mid-Cycle. Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-4: Room creation

An Underwriter can create a Room specifying Contribution amount, Cadence, and Member count. Realizes UJ-2.

**Consequences (testable):**
- Member count must be within the Underwriter's subscription tier capacity; a request above it reverts.
- Room creation requires posted Backstop capital meeting the adequacy rule in FR-19 before the Room accepts Members.
- V1 accepts weekly Cadence only; other values revert. `[ASSUMPTION: V1 restriction, not a permanent product limit]`

#### FR-5: Member admission

An Underwriter can review and admit or reject a prospective Member based on their Pal3 history. Realizes UJ-2.

**Consequences (testable):**
- The Underwriter view exposes: Trust Score, Cycles completed, Default count, account age. It does not expose name, documents, employer, or bank details.
- A rejected applicant's locked funds, if any, are returned in full.

#### FR-6: Room start

A Room starts automatically when it reaches its configured Member count, and only then.

**Consequences (testable):**
- Payout Positions are computed and frozen at start (see FR-11).
- Room parameters become immutable at start: no Member may be added or removed except through Stake exhaustion (FR-16), and no amount, Cadence, or ordering may change.
- If the Room does not fill within its open window, it is cancelled and every Stake and Contribution is returned in full. `[ASSUMPTION: open window of 14 days]`

#### FR-7: Room completion

A Room closes after its final Round, returning remaining Stakes.

**Consequences (testable):**
- Every Member who completed all Rounds has their full Stake returned.
- Trust Score cycle-completion credit is applied at close (FR-10).

### 4.3 Contributions and Escrow

**Description:** The contract holds every Contribution from the moment it is paid until it is released as a Pot. No human, including the Underwriter and the platform operator, has withdrawal authority. This is the structural removal of organizer fraud. Realizes UJ-1, UJ-4.

**Functional Requirements:**

#### FR-8: Scheduled contribution

A Member can pay their Contribution for the current Round, and the contract escrows it. Realizes UJ-1.

**Consequences (testable):**
- Contributions are accepted only for the current open Round.
- Escrowed funds are withdrawable by no party under any code path other than Payout (FR-12), Slash (FR-15), or cancellation refund (FR-6).
- A Member who has already contributed for the current Round cannot double-pay.

#### FR-9: Contribution status visibility

Any Member can see the contribution status of every Member in their Room for the current Round.

**Consequences (testable):**
- Status shows per Member: paid, pending, in Grace Window, or defaulted — identified by Member handle, not legal identity.

### 4.4 Trust Engine

**Description:** The Trust Score is the product's defining mechanism and its only portable asset. It is network-wide — earned in one Room and carried into every future one — and it drives both Payout Position and Stake size, so reliability compounds into tangible advantage. The score is computed off-chain by the trust service and committed on-chain at the points where it has consequences (Room join, Room start). Realizes UJ-1, UJ-3, UJ-4.

**A stated design position: reputation over collateral.** Inverse Stake sizing (FR-13) combined with trust-ranked ordering (FR-11) produces an inversion worth naming, because it is deliberate and a reviewer will otherwise find it unaided. The highest-trust Member is paid in Round 1, still owes every remaining Contribution, and posts the *smallest* Stake. The lowest-trust Member is paid last, owes nothing afterward, and posts the *largest*. Measured purely as collateral, that is backwards — the Stake covers roughly a tenth of what an early recipient still owes.

Pal3 accepts this knowingly. The product's central claim is that **identity-anchored reputation predicts repayment behavior better than collateral does** — which is precisely what pseudonymous on-chain ROSCAs could not do, and why they failed. Members are paid early *because* they are least likely to walk away; collateral is the secondary defense, and the Underwriter's Backstop is the third. Requiring early recipients to post collateral proportional to their remaining obligation would price most Members out of joining at all, which would defeat the product's purpose more surely than the risk it hedges.

The pilot is the test of that claim. If Defaults concentrate among high-trust, early-paid Members, the thesis is wrong and Stake sizing must be rebuilt against remaining obligation. `[NOTE FOR PM: this is the single most important thing the pilot can falsify]`

**Functional Requirements:**

#### FR-10: Trust Score computation

The system maintains a Trust Score of 0–1000 per Member, updated on defined events.

**Consequences (testable):**
- On-time Contribution: +10. `[ASSUMPTION]`
- Contribution cured within Grace Window: +2. `[ASSUMPTION]`
- Default: −150. `[ASSUMPTION]`
- Cycle completed with zero Defaults: +50. `[ASSUMPTION]`
- Room abandoned via Stake exhaustion: −400. `[ASSUMPTION]`
- Score is clamped to [0, 1000] after every update.
- Score changes are deterministic and reproducible from event history — the same event sequence always yields the same score.

#### FR-11: Payout ordering with Fairness Floor

The system assigns Payout Positions by descending Trust Score at Room start, subject to the Fairness Floor. Realizes UJ-1.

**Consequences (testable):**
- Members are ranked by Trust Score descending; the highest score receives the Pot in Round 1.
- Ties are broken deterministically and auditably by a published rule; no randomness is used. `[ASSUMPTION: tie broken by earlier Room-join timestamp]`
- A Member assigned the final Payout Position in the two immediately preceding Cycles *within the same Room* cannot be assigned it a third consecutive time; they swap with the next-lowest-ranked eligible Member.
- Assigned positions are visible to every Member before the Room starts, and frozen thereafter.

#### FR-12: Pot payout

The contract releases the full Pot to the Member holding the current Round's Payout Position. Realizes UJ-4.

**Consequences (testable):**
- The recipient receives the full Pot — Member count × Contribution amount — with no platform deduction.
- Payout executes on the scheduled Round boundary regardless of Default events, having drawn on Slash and Backstop as needed (FR-15, FR-16).
- Each Member receives exactly one Payout per Cycle.

#### FR-13: Stake sizing

The system computes a Member's required Stake inversely to their Trust Score at the moment of joining. Realizes UJ-1.

**Consequences (testable):**
- Stake = Contribution × (2 − TrustScore/1000), yielding 2× Contribution at score 0 and 1× at score 1000. `[ASSUMPTION]`
- The computed Stake is displayed to the Member before they commit, with the score that produced it.
- Stake is locked at join and is not recomputed mid-Cycle even if the Member's score changes.

### 4.5 Default Handling

**Description:** The waterfall that makes the promise true: a missed Contribution never costs the scheduled recipient anything. Grace first, because a late salary is not the same as walking away. Then the defaulter's own Stake. Then the Underwriter's Backstop. The recipient is paid in full at every branch. Realizes UJ-3.

**Functional Requirements:**

#### FR-14: Grace Window

A Member who misses a Contribution deadline can still pay within a 48-hour Grace Window without incurring a Default. Realizes UJ-3.

**Consequences (testable):**
- The Round does not advance and no Payout occurs while a Grace Window is open.
- The Member is notified at Grace Window open and at 24 hours remaining.
- A Contribution paid within the window is accepted in full and recorded as cured, applying the FR-10 cured-contribution score change rather than a Default penalty.
- Other Members see the Room state as "awaiting contribution" with time remaining.

#### FR-15: Slash and Backstop waterfall

On Default, the contract covers the shortfall from the defaulter's Stake first, then the Underwriter's Backstop. Realizes UJ-3.

**Consequences (testable):**
- The Slash amount equals exactly the missed Contribution — not the whole Stake.
- If the remaining Stake is less than the missed Contribution, the entire remaining Stake is slashed and the Backstop covers the difference.
- The scheduled recipient receives the full Pot in the same Round, with no delay and no reduction, in every branch.
- The defaulting Member's Trust Score takes the FR-10 Default penalty.
- The Round advances immediately on resolution.

#### FR-16: Stake exhaustion and removal

A Member whose Stake is fully depleted is removed from the Room, and the Backstop carries their remaining obligations.

**Consequences (testable):**
- Removal occurs at the moment a Slash leaves the Stake at zero.
- The Backstop covers that Member's Contributions for all remaining Rounds in the Cycle.
- A removed Member who has not yet received their Payout does not receive one; a removed Member who already received it retains it, and the Backstop absorbs the loss.
- The removed Member's Trust Score takes the abandonment penalty (FR-10).

#### FR-17: Underwriter exposure visibility

An Underwriter can see their current and maximum possible exposure per Room at any time. Realizes UJ-2.

**Consequences (testable):**
- Maximum exposure is computable and displayed before the Underwriter commits to a Room.
- Live exposure updates on every Slash and Backstop draw.

### 4.6 Underwriter Subscription and Capacity

**Description:** Pal3's revenue, and simultaneously a risk control. Subscription tier gates how many concurrent Rooms an Underwriter may run and how large each may be — but capacity unlocks only against posted Backstop capital, so a tier upgrade can never outrun solvency. The platform never touches pooled Member funds. Realizes UJ-2.

**Functional Requirements:**

#### FR-18: Tiered capacity

An Underwriter's subscription tier determines their maximum concurrent Rooms and maximum Members per Room.

**Consequences (testable):**
- A Room creation request exceeding either limit reverts.
- V1 operates a single tier: one Room, maximum 10 Members. `[ASSUMPTION: pilot restriction]`

#### FR-19: Capital adequacy gate

Room capacity is gated on posted Backstop capital, not on subscription payment alone.

**Consequences (testable):**
- Required Backstop is a published function of Member count and Contribution amount. `[ASSUMPTION: minimum Backstop = 2 × Contribution × Member count, i.e. cover for two total defaults]`
- A Room cannot open Member admission until the required Backstop is posted and locked.
- Backstop capital cannot be withdrawn while any Room it collateralizes is in progress.

#### FR-20: Per-contribution fee

An Underwriter earns a fee on each Contribution, disclosed to Members before they join.

**Consequences (testable):**
- The fee is displayed as both a percentage and an absolute per-Round amount on the Room terms screen before a Member commits.
- The fee is charged on Contribution, not on Payout — a Member's Pot is never reduced.
- The platform takes no share of the fee and no share of the Pot.

### 4.7 Member Application

**Description:** The mobile-first surface. Its job is to make a crypto-backed product legible to someone who has never held a stablecoin and whose main prior experience of paluwagan may be losing money. Realizes UJ-1, UJ-3, UJ-4.

**Functional Requirements:**

#### FR-21: Wallet connection

A Member can connect a Stellar wallet from a mobile device and fund it with the Room's stablecoin.

**Consequences (testable):**
- Wallet connection is implemented through **WalletConnect**, not through a browser-extension API. This is a mobile-first requirement, not a preference: Soroban contract invocation requires `signAuthEntry`, and that path against Freighter Mobile must go through WalletConnect directly. An extension-only integration would work on desktop and fail on the product's primary surface.
- At least one WalletConnect-compatible Stellar wallet is supported end to end. `[ASSUMPTION: Freighter via WalletConnect for V1]`
- A Member with insufficient balance for Stake plus first Contribution is told the exact shortfall before attempting to join.

#### FR-22: Room terms disclosure

A Member sees complete Room terms before committing any funds. Realizes UJ-1.

**Consequences (testable):**
- One screen shows: Contribution amount, Cadence, Member count, their Payout Position and date, their Stake, the Underwriter's fee, and the default-handling waterfall in plain language.
- The screen states explicitly that funds are denominated in a USD-pegged stablecoin and not in pesos, and that the peso value may move over the Cycle.
- A Member must acknowledge the FX disclosure before joining. This acknowledgement is recorded.

#### FR-23: Schedule and status view

A Member can see their Room's full schedule and current state at any time.

**Consequences (testable):**
- Shows all Rounds, all Payout Positions and dates, and which Rounds are complete.
- Shows their own Trust Score and its change history.

## 5. Non-Goals (Explicit)

- **Pal3 is not a lender.** No credit is extended to anyone; a Member never receives more than the group contributed.
- **Pal3 is not a yield product.** Members earn no return. This is a deliberate regulatory position, not a missing feature — see §8.
- **Pal3 is not a custodian.** The platform holds no Member funds and has no withdrawal authority over escrowed value at any point.
- **Pal3 is not an exchange or on/off-ramp.** Members arrive with a funded wallet in V1.
- **Pal3 does not adjudicate disputes in V1.** There is no appeals process, no manual override, and no admin key that can reverse a Slash.
- **Pal3 is not becoming a general DeFi protocol.** No lending, no staking rewards, no token, no secondary market in positions or Trust Scores.

## 6. MVP Scope

### 6.1 In Scope

- One Room type: fixed Contribution, weekly Cadence, 5–10 Members, one Underwriter.
- Soroban contracts: escrow, Stake lock, Slash, Backstop draw, scheduled Payout.
- Trust Engine: score computation, Stake sizing, Payout ordering with Fairness Floor.
- Default waterfall: Grace Window → Slash → Backstop, recipient paid in full at every branch.
- KYC via third-party provider; Pal3 holds attestation only.
- Member mobile app: wallet connect, join, contribute, schedule and status, Trust Score view.
- Underwriter view: Member admission, exposure, Room monitoring.
- FX disclosure and recorded acknowledgement.

### 6.2 Out of Scope for MVP

- **Peso-stablecoin rail** — depends on a BSP-sanctioned peso asset existing on Stellar. Phase 2.
- **Fiat on/off-ramp** — Members arrive with funded wallets. Phase 2.
- **Open public enrolment** — V1 is a closed, recruited pilot.
- **Underwriter subscription billing** — V1 runs a single tier with no billing automation; the tier model is specified but not implemented. `[NOTE FOR PM: revenue mechanism is unexercised in V1 — the SaaS thesis remains untested after the pilot]`
- **Multi-Room trust-graph effects** — the score is network-wide by design but V1 has one Room, so cross-Room accrual is unexercised.
- **Employer-sponsored rooms, multi-currency, secondary markets** — vision, not V1.
- **Dispute resolution and legal recovery** — no UI, no process. `[NOTE FOR PM: a Member who believes a Slash was wrong has no recourse in V1. Acceptable for a consenting pilot cohort; not acceptable at public launch]`

## 7. Success Metrics

**Primary**
- **SM-1: Cycle completion.** One Room completes all Rounds with every Pot paid in full and on schedule. Target: 1 completed Cycle. Validates FR-6, FR-8, FR-12.
- **SM-2: Default waterfall proven.** At least one Default occurs (induced if it does not arise naturally) and resolves through the full waterfall with the recipient paid in full and on time. Target: ≥1 resolved Default, zero recipient loss. Validates FR-14, FR-15, FR-16.
- **SM-3: Zero loss to any scheduled recipient.** No Member receives less than the full Pot on their Round, under any circumstance. Target: 0 shortfalls. Validates FR-12, FR-15.

**Secondary**
- **SM-4: Retention into a second Cycle.** Members who complete the pilot opt into another Room. Target: ≥60% of the cohort. `[ASSUMPTION: target figure]` Validates the value proposition, not a single FR.
- **SM-5: Trust Score accrual is legible.** Members can correctly state their score and why it changed when asked. Target: ≥80% of the cohort. Validates FR-10, FR-23.
- **SM-6: Underwriter economics hold.** Fees collected exceed Backstop draws across the Cycle. Validates FR-19, FR-20.

**Counter-metrics (do not optimize)**
- **SM-C1: Default rate.** A *zero* default rate in the pilot is a warning, not a triumph — it means the waterfall went unexercised and SM-2 was satisfied only artificially. Counterbalances SM-3.
- **SM-C2: Trust Score inflation.** Average cohort score rising quickly indicates the score is too easy to earn and will not discriminate risk at scale. Counterbalances SM-5.
- **SM-C3: Room size.** Resist increasing Member count to make the pilot look more substantial. Larger Rooms mean longer Cycles and greater Underwriter exposure, both of which raise the cost of learning. Counterbalances SM-1.

## 8. Compliance and Regulatory

The regulatory position is a *design* position, and it is load-bearing enough to belong in the specification rather than an appendix.

**The member-side argument.** The SEC's stated objection to "Online Paluwagan" schemes is the *promised return* — advisories cite 10%–757% over 1–90 days — which is what makes them investment contracts. Pal3 promises Members nothing beyond their own money returned on schedule (§5, FR-12, FR-20). Member-side investment-contract exposure is therefore structurally near-zero, and the analysis narrows to the Underwriter relationship alone.

**Open exposures, in priority order:**
1. **SEC — investment contract.** The Underwriter earns a premium from a pool. Needs PH fintech counsel.
2. **Insurance Commission — surety.** Whether the Backstop constitutes insurance or suretyship.
3. **BSP — VASP licensing.** New VASP licences were frozen in Sept 2025; production likely requires operating on a licensed VASP/EMI rail.
4. **AMLC — KYC/AML obligations**, discharged through the KYC Provider.
5. **NPC — Data Privacy Act 2012**, addressed in §9.

**V1 posture:** a closed pilot with a consenting cohort, no public solicitation, no promised return, no platform custody of funds. `[ASSUMPTION: this posture is defensible for a pilot without prior counsel sign-off — to be confirmed by PH fintech counsel, funded as milestone T3]`

## 9. Data Governance and Privacy

Pal3 processes personal data of Philippine citizens and is subject to the Data Privacy Act of 2012 and National Privacy Commission oversight.

- **Minimization by architecture.** Identity documents, employment records, and bank details are held solely by the KYC Provider. Pal3 holds an attestation and an opaque reference (FR-1). Pal3 is not a controller of document data.
- **On-chain data.** No personal data is written on-chain. Wallet addresses, Trust Scores, and Room state are pseudonymous; the link to a legal identity exists only in the KYC Provider's records.
- **Cross-Member visibility.** Members see other Members' contribution status by handle; Underwriters see Trust Score and history. Neither sees legal identity (FR-5, FR-9).
- **Irreversibility disclosure.** Trust Score history is append-only and a Default is permanent in the event history. Members must be told before joining that this record cannot be erased. `[NOTE FOR PM: an append-only reputation record sits in tension with the Data Privacy Act's rectification and erasure rights. This needs counsel input and may require the score to be recomputable from an amendable event store rather than an immutable chain record]`

## 10. Cross-Cutting NFRs

- **Funds safety is the dominant quality attribute.** Contracts holding escrow, Stake, and Backstop must pass an audit-readiness review before any mainnet Room opens with real value (brief milestone T2). No mainnet pilot begins without it.
- **Determinism.** Payout ordering, Slash amounts, and Trust Score changes must be reproducible from event history by any observer. No randomness anywhere in the system.
- **No privileged override.** No admin key, upgrade path, or operator action can move escrowed funds, alter a Payout Position mid-Cycle, or reverse a Slash. This constraint is what makes the organizer-fraud claim true, and it outranks operational convenience.
- **Availability at Round boundaries.** Contribution and Payout must succeed at scheduled boundaries. Off-chain trust-service unavailability must never block an on-chain Payout — scores are committed on-chain at Room start precisely so payout does not depend on a live service.
- **Cost per Round** must stay negligible relative to a ₱1,000-equivalent Contribution; transaction cost is a product constraint, not just an engineering one.

## 11. Monetization

- **Platform:** SaaS subscription from Underwriters, tiered by concurrent Room count and per-Room capacity (FR-18), gated on capital adequacy (FR-19). The platform never enters the pooled-money flow — software, not a financial counterparty.
- **Underwriters:** per-Contribution fee (FR-20), the risk premium for covering Defaults.
- **Members:** no yield (§5, §8).

`[NOTE FOR PM: V1 implements no billing. The subscription thesis — that Underwriters will pay a recurring fee for capacity — is the least-tested assumption in the business model, and the pilot does not test it]`

## 12. Platform and Constraints

- **Surface:** mobile-first web application, with wallet connection via WalletConnect (FR-21). `[ASSUMPTION: PWA rather than native for V1, to avoid app-store review in the eight-week window]`
- **Chain:** Stellar/Soroban. Rust/WASM contracts for escrow, Stake, Slash, Payout.
- **Asset:** USD-pegged stablecoin. No peso rail in V1 (§6.2).
- **Trust Engine placement:** computed off-chain as service code, committed on-chain at consequence points, deliberately chain-agnostic so the mechanism remains portable if Pal3 later migrates to reach a peso rail.
- **No verifiable randomness dependency.** Soroban has no native VRF, and the design removes the need for one — ordering is fully deterministic (FR-11).
- **Open source.** All Soroban contracts are published under a permissive licence (Apache 2.0 or MIT) from first deployment. For a product asking people to escrow money with strangers, public, auditable contract source is part of the trust proposition, not a concession. `[ASSUMPTION: Apache 2.0 over MIT, for the explicit patent grant]`

**Why Stellar, stated for the record.** The dependency is economic, not incidental. A ₱1,000-equivalent weekly Contribution cannot absorb meaningful transaction fees — a $0.50 fee on an ~$18 Contribution is close to 3% per Round, charged against a product whose proposition is that it costs less than the free alternative it replaces. Stellar's fee structure is what makes weekly micro-Contributions viable at all, and the NFR above is a direct consequence. Fast, predictable finality matters for the same reason: Rounds settle on fixed boundaries. The anchor network is additionally the realistic route to the Phase 2 peso rail.

The Trust Engine is nonetheless built as portable service code (above). That is a deliberate risk hedge against the Phase 2 peso-rail dependency, not a statement that the chain is interchangeable — the contracts holding every peso of Member value are Soroban-native, and the unit economics do not work on a high-fee chain.

## 13. Open Questions

Triaged 2026-08-08. None blocks architecture or the eight-week build; each carries an owner and a revisit condition.

**Resolve before the mainnet pilot opens with real value:**

1. **Grace Window abuse.** A Member could cure inside the Grace Window every single Round — taking the +2 rather than the −150, and delaying every Payout by up to 48 hours indefinitely. FR-14 has no escalation on repeated use. *Owner: founder. Revisit: before pilot recruitment. Likely fix: after N cures in one Cycle, treat the next as a Default.*
2. **Backstop adequacy.** Verify the FR-19 formula against a full exposure table, including the worst case of the Round-1 recipient defaulting immediately after Payout. *Owner: founder. Revisit: before posting Backstop as pilot Underwriter.*
3. **Counsel confirmation** on the §8 regulatory posture and the §9 erasure-rights tension. *Owner: PH fintech counsel, funded as milestone T3. Revisit: post-award.*
4. **Named pilot cohort.** Size is settled at 5–10; the actual people are not identified. *Owner: founder. Revisit: during the build window, not after it.*

**Deferred past V1 — structurally absent because V1 has one Room and one Underwriter:**

5. **Underwriter lapse mid-Cycle.** What happens to an in-flight Room if the Underwriter's subscription lapses or the Backstop is drawn to zero? *Deferred: the founder is the V1 Underwriter, so the scenario cannot arise. Revisit: before any third-party Underwriter onboards.*
6. **Trust Score decay.** Without decay, a score earned in 2026 still commands a low Stake in 2030 on stale evidence. *Deferred: no Member can be inactive within a single-Cycle pilot. Revisit: before the second cohort.*
7. **Rehabilitation.** A Default is currently a permanent −150 with no defined path back. *Deferred: no rehabilitation horizon exists inside one Cycle. Revisit: before the second cohort.*
8. **Fairness Floor across Rooms.** FR-11 scopes the floor within a Room; a persistently low-scoring Member could still be placed late in every Room they join. *Deferred: V1 has one Room. Revisit: when a Member can hold concurrent Rooms.*
9. **Sybil resistance beyond KYC.** FR-3 relies entirely on the KYC Provider's duplicate detection. *Deferred: a recruited cohort of known individuals cannot be Sybil-attacked. Revisit: before open enrolment.*

## 14. Assumptions Index

Every `[ASSUMPTION]` in this document, for explicit confirmation:

- §4.1 FR-2 — Basic verification starts a Member at Trust Score 300; full verification at 400.
- §4.2 FR-4 — V1 accepts weekly Cadence only.
- §4.2 FR-6 — Room open window is 14 days before cancellation.
- §4.4 FR-10 — All five score deltas: +10 on-time, +2 cured, −150 Default, +50 Cycle completed, −400 abandonment.
- §4.4 FR-11 — Ties broken by earlier Room-join timestamp.
- §4.4 FR-13 — Stake formula: Contribution × (2 − TrustScore/1000).
- §4.6 FR-18 — V1 runs a single tier: one Room, maximum 10 Members.
- §4.6 FR-19 — Minimum Backstop = 2 × Contribution × Member count. **Note:** this covers roughly two total Defaults in a 10-Member Room, because a Member defaulting immediately after Payout leaves ~9 Contributions outstanding against a Stake covering ~2. Verify against the exposure table before the pilot opens.
- §4.7 FR-21 — Freighter, reached via WalletConnect, as the V1 wallet.
- §7 SM-4 — 60% retention target.
- §8 — Closed-pilot posture is defensible without prior counsel sign-off.
- §12 — PWA rather than native for V1.
