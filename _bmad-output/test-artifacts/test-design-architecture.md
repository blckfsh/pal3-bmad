---
workflowStatus: 'completed'
totalSteps: 5
stepsCompleted:
  - 'step-01-detect-mode'
  - 'step-02-load-context'
  - 'step-03-risk-and-testability'
  - 'step-04-coverage-plan'
  - 'step-05-generate-output'
lastStep: 'step-05-generate-output'
nextStep: ''
lastSaved: '2026-08-13'
workflowType: 'testarch-test-design'
inputDocuments:
  - '_bmad-output/planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md'
  - '_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md'
  - '_bmad-output/planning-artifacts/epics.md'
  - '_bmad-output/planning-artifacts/implementation-readiness-report-2026-08-12.md'
---

# Test Design for Architecture: Pal3 V1

**Purpose:** Architectural concerns, testability gaps, and NFR requirements for review. Serves as the contract between test design and implementation on what must be resolved before test development begins.

**Date:** 2026-08-13
**Author:** Murat (TEA Master Test Architect)
**Status:** Architecture Review Pending
**Project:** paluwagan3
**PRD Reference:** `planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md`
**ADR Reference:** `planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md` (AD-1…AD-17)

---

## Executive Summary

**Scope:** System-level test design for the Pal3 V1 pilot — Registry, Factory and Room contracts on Stellar/Soroban, the off-chain trust, KYC, indexer, rates and API services, and the member/underwriter PWA. Covers all 5 epics and 45 stories.

**Business Context** (from PRD):

- **Problem:** Philippine paluwagan savings circles fail through organizer fraud and member default. Pal3 removes the organizer's custody of funds structurally and covers default through a Stake-then-Backstop waterfall.
- **Impact:** Pilot success is defined as one Room completing every Round with every Pot paid in full (SM-1), at least one Default resolving through the full waterfall (SM-2), and zero shortfall to any scheduled recipient (SM-3).
- **Timeline:** Eight-week build window; testnet then mainnet pilot, gated on an audit-readiness review (milestone T2).

**Architecture** (from the spine, 17 decisions):

- **AD-1 / AD-7:** Ledger-authoritative hexagonal — contracts hold all state and all value; off-chain components observe or submit but never adjudicate.
- **AD-6 / AD-9 / AD-11:** No privileged path to member value; the default waterfall resolves atomically in one invocation; no randomness anywhere.
- **AD-2 / AD-15:** Registry + Factory + per-Room instance topology; every entry point authorizes the transaction source with a single `require_auth`.
- **Stack:** Rust 1.84+ / soroban-sdk 25.0.0 (`wasm32v1-none`); TypeScript services; React + Vite PWA; Stellar Wallets Kit over WalletConnect.

**Expected Scale:** One Room, one Underwriter, 5–10 Members, weekly Cadence, single Cycle. Scale is deliberately small (counter-metric SM-C3 resists growing it) — this system's difficulty is correctness, not load.

**Risk Summary:**

- **Total risks:** 21 — 1 critical (score 9), 11 high (score ≥6), 9 medium, 0 low
- **Test effort:** ~124 scenarios, ~175–280 hours (~4.5–7 weeks, one engineer full-time)

---

## Quick Guide

### BLOCKERS — Must Be Decided (Cannot Proceed Without)

Pre-implementation critical path. These block test development, not just test execution.

1. **R-001: Backstop adequacy formula is insufficient against its own worst case** — the PRD states the arithmetic itself: a Member defaulting immediately after Payout leaves ~9 Contributions outstanding against a Stake covering ~2, while the minimum Backstop covers only two total Defaults. Until the formula is corrected or the exposure bound proven, the "zero loss to any scheduled recipient" claim has no evidence behind it. (Owner: Founder/product — before mainnet, and before Story 5.5 can be written to a target)
2. **R-010: KYC provider is unnamed** — no provider is identified in the PRD, spine, or assumptions index, so the webhook payload, duplicate-detection signal, tier vocabulary, and idempotency semantics are all undefined. Stories 1.6 and 1.8 cannot be built to a real contract, and a stub built now encodes guesses. (Owner: Founder — before Epic 1 Story 1.6)
3. **R-011: No wallet test-signer seam** — every money-moving PWA action is signed through WalletConnect, whose pairing flow is human-in-the-loop and not automatable. Architecture must define a test-only signer satisfying the same interface, or no automated end-to-end coverage of join/contribute/payout is possible at all. (Owner: Founder/Dev — before Story 1.2)
4. **R-012: No chain-state seeding harness** — reaching states like "Round 5, one Member in Grace, one removed by Stake exhaustion" requires driving real invocations with ledger-time control. Free inside Rust contract tests; absent for API and PWA testing against a deployed network. (Owner: Founder/Dev — before Story 1.2)
5. **U-5 / R-009: Stake rounding semantics unspecified** — `Contribution × (2 − TrustScore/1000)` is real-valued, but the conventions mandate integer stroops and forbid floating point. Rounding direction and operation order must be specified, not chosen at implementation time. A test cannot assert a value the specification does not determine. (Owner: Founder/product — before Story 2.5)

**What is needed:** these five resolved pre-implementation, or test development is blocked. Items 2–4 gate Story 1.2 specifically, which is the second story in the sprint plan.

---

### HIGH PRIORITY — Should Be Validated (Recommendation Provided)

1. **R-002 / R-003 / R-005: The three claim-bearing invariants need dedicated negative suites, not incidental coverage.** Recommendation: treat "no privileged path to escrow" (AD-6), "waterfall is atomic" (AD-9), and "no personal data on-chain" (AD-10) as exhaustive negative test matrices — every entry point × every caller class — rather than as properties assumed from design. These three are what the product's central claims rest on. (Implementation phase, contracts)
2. **R-007: AD-15 conformance should be enforced statically, not discovered late.** Recommendation: add a CI check asserting no entry point requires a detached authorization entry or multi-party authorization. Discovering a violation after the PWA is built means rearchitecting the primary surface inside an eight-week window. (Implementation phase)
3. **R-004: Trust Score reproduction needs a genuinely independent implementation.** Recommendation: Story 4.5's reproduction harness must not share code with `services/trust`, or it proves only that the code agrees with itself. Non-determinism from map iteration order or an ambient clock is the specific failure mode. (Implementation phase)
4. **R-006: TTL and archival behaviour will pass in test and can still fail on-network.** Recommendation: schedule a deliberate stalled-Room soak on testnet before the mainnet gate; the spine's Deferred list already flags that a stalled Room has no cheap rescue path and proposes a permissionless `extend_room_ttl`. Decide whether that lands in V1. (Before testnet gate)
5. **R-008: The reputation-over-collateral thesis is the pilot's designated falsification target.** The PRD names it as the single most important thing the pilot can falsify. Recommendation: instrument it — record, per Default, the defaulter's Trust Score and Payout Position, so the concentration question is answerable from pilot data rather than argued afterward. This is a measurement requirement, not a code fix. (Implementation phase)
6. **U-1: "Negligible" is not a threshold.** Story 5.4 measures transaction cost against nothing. Recommendation: ratify a number — a maximum percentage of Contribution, or an absolute ceiling — so the story has a pass/fail criterion. (Before Story 5.4)

---

### INFO ONLY — Solutions Provided

1. **Test level split:** contract tests (Rust/`Env`) carry all money-moving logic; service integration for indexer replay and score computation; a deliberately thin E2E set for journeys only. Rationale: authoritative state is on-chain, so that is where the invariants belong.
2. **Coverage:** ~124 scenarios, prioritized P0–P3 with explicit risk linkage. P0 runs at ~18% rather than the usual <10% — a deliberate, recorded deviation, because the P0 set is almost entirely invariants rather than features.
3. **Execution:** PR / Nightly / Weekly. Everything functional runs per PR; only the exposure-table sweep, cost measurement, fuzzing, and the outage drill defer.
4. Detail lives in the companion QA document (`test-design-qa.md`). No decisions needed here.

---

## For Architects and Devs — Open Topics

### Risk Assessment

**Total risks identified:** 21 (1 critical, 11 high with score ≥6, 9 medium, 0 low)

#### Critical Risk (Score 9) — GATE BLOCKER

| Risk ID | Category | Description | Probability | Impact | Score | Mitigation | Owner | Timeline |
|---|---|---|---|---|---|---|---|---|
| **R-001** | **BUS** | Minimum Backstop (2 × Contribution × Member count) is insufficient against the PRD's own stated worst case — a Member defaulting immediately after Payout leaves ~9 Contributions outstanding against a Stake covering ~2. Backstop exhaustion means a scheduled recipient is paid short, contradicting SM-3 and the product's central promise. | 3 | 3 | **9** | Build the full exposure table across every default pattern for a 10-Member Room; correct the formula or cap Room parameters so no pattern produces a shortfall; prove it by simulation before capital is posted | Founder (product) | Before mainnet pilot; before Story 5.5 |

#### High-Priority Risks (Score ≥6) — Immediate Attention

| Risk ID | Category | Description | Probability | Impact | Score | Mitigation | Owner | Timeline |
|---|---|---|---|---|---|---|---|---|
| **R-002** | **SEC** | A privileged path to escrow is introduced — an admin, Underwriter, or upgrade entry point that can move funds, alter a Payout Position after start, or reverse a Slash. Destroys the organizer-fraud claim entirely (AD-6). | 2 | 3 | **6** | Exhaustive negative-authorization matrix over every entry point × every caller class; audit-readiness review as the backstop gate | Founder/Dev | Before testnet |
| **R-003** | **TECH** | The waterfall is not truly atomic — Slash succeeds, Payout does not, leaving the recipient short and the Room inconsistent (AD-9). | 2 | 3 | **6** | Fault-injection tests on every leg asserting full revert; assert no Round advances until the recipient holds the full Pot | Founder/Dev | Before testnet |
| **R-004** | **DATA** | Trust Score is not independently reproducible — non-determinism (map iteration order, ambient clock, unversioned function change) yields different scores from the same event history. Ordering and Stake sizing both derive from it, so a wrong score changes who is paid first and how much collateral they post. | 2 | 3 | **6** | Independent reproduction harness sharing no code with `services/trust` (Story 4.5); version recorded with every commit; replay-equality tests | Founder/Dev | Before Epic 4 close |
| **R-005** | **SEC** | Personal data reaches the chain — a provider reference that is actually derived from an identity document, or a KYC webhook field written through to Registry. Irreversible publication of Philippine citizens' identity data under the Data Privacy Act (AD-10, §9). | 2 | 3 | **6** | Field-level allowlist on everything the KYC adapter writes; contract-side assertion that Registry stores only address, opaque reference, tier, integers; review gate on any Registry schema change | Founder/Dev | Before testnet |
| **R-006** | **TECH** | Soroban state archival renders a Room unusable mid-Cycle. AD-3 extends TTL on every write, which covers weekly activity, but a Room stalled in Grace or the open window has no rescue path — the spine concedes this in its Deferred list. | 2 | 3 | **6** | Decide whether permissionless `extend_room_ttl` lands in V1; stalled-Room soak on testnet rather than in `Env` alone | Founder/Dev | Before mainnet pilot |
| **R-007** | **TECH** | An entry point is designed requiring detached auth or multi-party authorization, violating AD-15 and falling outside Stellar Wallets Kit's WalletConnect module — discovered late, the primary mobile surface becomes unbuildable. | 2 | 3 | **6** | Static CI check that no entry point requires a detached authorization entry; assert single `require_auth` on transaction source at every entry point | Founder/Dev | Before Story 1.7 |
| **R-008** | **BUS** | The reputation-over-collateral thesis is wrong — Defaults concentrate among high-trust, early-paid Members who post the smallest Stakes and owe the most. The PRD names this as the single most important thing the pilot can falsify. | 2 | 3 | **6** | Instrument every Default with the defaulter's Trust Score and Payout Position so concentration is measurable; Backstop absorbs the pilot cost either way | Founder (product) | Pilot measurement |
| **R-009** | **DATA** | Stake rounding is unspecified — a real-valued formula over an integer-stroop domain with no stated rounding direction or operation order. Two faithful implementations disagree, and no test can assert an undetermined expected value. | 3 | 2 | **6** | Specify rounding direction and operation order in the PRD; boundary tests across the full 0–1000 score domain | Founder (product) | Before Story 2.5 |
| **R-010** | **OPS** | KYC provider is unnamed, so the webhook contract, duplicate-detection signal, tier vocabulary, and idempotency semantics are undefined. Stories 1.6 and 1.8 cannot be built to a real contract; a stub built now encodes guesses the real provider will contradict. | 3 | 2 | **6** | Name the provider, or freeze an explicit adapter interface and build the stub to that interface with the guesses recorded as assumptions | Founder | Before Story 1.6 |
| **R-011** | **OPS** | No wallet test-signer seam. WalletConnect pairing is human-in-the-loop and not automatable, so every money-moving end-to-end journey is manual-only. | 3 | 2 | **6** | Define a test-only signer satisfying the same interface, injectable by build flag; keep it structurally impossible to enable in production builds | Founder/Dev | Before Story 1.2 |
| **R-012** | **OPS** | No chain-state seeding harness. Mid-Cycle states are unreachable for API and PWA testing without scripted invocation sequences plus ledger-time control. | 3 | 2 | **6** | Build a seeding harness alongside the test framework; script named fixture states (filling, started, grace-open, default-resolved, member-removed, closed) | Founder/Dev | Before Story 1.2 |

#### Medium-Priority Risks (Score 3–5)

| Risk ID | Category | Description | Probability | Impact | Score | Mitigation | Owner |
|---|---|---|---|---|---|---|---|
| R-013 | BUS | Grace Window abuse — a Member cures every Round, taking +2 instead of −150 and delaying every Payout by up to 48 hours indefinitely. FR-14 has no escalation. Open Question 1. | 2 | 2 | 4 | Escalation rule after N cures per Cycle; simulate repeated-cure sequences | Founder (product) |
| R-014 | TECH | The AD-5 score-snapshot guard is missing or incorrect, so a score written between display and submission silently changes the terms a Member agreed to. | 2 | 2 | 4 | Explicit revert test: join carrying a stale score value must fail | Founder/Dev |
| R-015 | OPS | AD-7's claim that a total off-chain outage leaves in-flight Rooms fully operable is asserted but never exercised. | 2 | 2 | 4 | Outage drill: services down, complete a full Round by direct invocation | Founder/Dev |
| R-016 | PERF | Per-Round transaction cost exceeds the product constraint. AD-9's atomic waterfall concentrates many storage writes into one invocation, and no cost ceiling is defined (U-1). | 2 | 2 | 4 | Ratify a threshold; measure the worst-case waterfall invocation, not the happy path | Founder/Dev |
| R-017 | DATA | Indexer read models drift from chain state and Members act on wrong information. No staleness bound is defined (U-3). | 2 | 2 | 4 | Replay-equality tests; define a staleness bound and surface it | Founder/Dev |
| R-018 | TECH | The Fairness Floor (FR-11) is structurally unexercisable in V1 — it triggers on the two preceding Cycles within a Room, and V1 runs one Room for one Cycle. It ships to cohort 2 with zero production evidence. | 3 | 1 | 3 | Simulated multi-Cycle unit tests constructing the precondition synthetically | Founder/Dev |
| R-019 | SEC | The trust service writes to Registry under a single operator key (spine Deferred item). Compromise lets an attacker set arbitrary scores, controlling payout ordering and Stake sizing network-wide. | 1 | 3 | 3 | Accepted for a single-operator pilot; revisit before any third-party Underwriter onboards | Founder |
| R-020 | TECH | FX provider unselected and the freshness window value undefined (U-4). Presentational only under AD-16, so the failure mode is a missing peso line, not a wrong amount. | 3 | 1 | 3 | Select provider and ratify the window before Story 2.8; test the omission path explicitly | Founder |
| R-021 | SEC | External token-contract (SAC) calls occur inside the atomic waterfall invocation. Soroban's model reduces classic reentrancy, but the external call surface is real. | 1 | 3 | 3 | Covered by the audit-readiness review; assert state consistency across the token call boundary | Founder/Dev |

#### Risk Category Legend

- **TECH**: Technical/architecture — flaws, integration, fragility
- **SEC**: Security — access control, authorization, data exposure
- **PERF**: Performance — cost, latency, resource limits
- **DATA**: Data integrity — loss, corruption, inconsistency, non-reproducibility
- **BUS**: Business impact — logic errors, product-thesis failure, user harm
- **OPS**: Operations — deployment, configuration, testability infrastructure

---

### NFR Testability Requirements

What architecture must provide so NFR validation can be automated later. Planning guidance, not evidence assessment.

| NFR Category | Threshold / Requirement | Current Design Support | Gap / Decision Needed | Planned Evidence |
|---|---|---|---|---|
| Security — funds safety | No privileged path to escrow, Positions, or Slash reversal (binary) | Supported — AD-6 states it as an invariant | Needs enforcement as an exhaustive negative matrix, not an assumed property | Negative-authorization suite; audit-readiness review (Story 5.7) |
| Security — privacy | No personal data on-chain (binary) | Supported — AD-10 | Field-level allowlist on the KYC adapter is undefined; provider unnamed (R-010) | Registry schema assertions; adapter allowlist tests |
| Reliability — atomicity | Waterfall fully succeeds or fully reverts; recipient always holds the full Pot | Supported — AD-9 | Fault injection points must be reachable in test | Fault-injection contract tests |
| Reliability — determinism | Every outcome reproducible from chain history by any observer | Strongly supported — AD-11 forbids randomness; AD-4 makes time an input | Reproduction harness must be genuinely independent (R-004) | Independent reproduction output (Story 4.5); replay-equality results |
| Reliability — liveness | Rooms remain operable during total off-chain outage | Supported — AD-7 and AD-13 (permissionless advance) | Never exercised (R-015); availability target undefined (U-2) | Outage drill report |
| Reliability — durability | Value-bearing state persistent, TTL extended on every write; expiry never control flow | Supported — AD-3, AD-4 | Archival behaviour under a stalled Room not faithfully reproducible in `Env` (R-006) | Testnet stalled-Room soak |
| Performance — cost | Cost per Round negligible against a ₱1,000-equivalent Contribution | Partial — Stellar chosen for exactly this reason | **Threshold UNKNOWN (U-1)** — no number to measure against | Measured cost report (Story 5.4) |
| Compliance | DPA 2012 minimization; SEC/BSP/IC posture | Supported by architecture (AD-10, no platform custody) | Counsel confirmation outstanding (milestone T3); §9 erasure-rights tension unresolved | Counsel sign-off; audit-readiness review |
| Maintainability | Bindings generated, never hand-written; one handle implementation | Supported — AD-8, AD-17 | Coverage target unratified (U-6) | CI binding-drift check; coverage reports |
| Availability — API/indexer | Read-model freshness bound | Not addressed | **UNKNOWN (U-3)**; hosting explicitly deferred in the spine | Deferred with hosting decision |

**Unknown thresholds:** U-1 transaction cost ceiling; U-2 Round-boundary availability target (may be N/A given AD-13 — confirm); U-3 read-model staleness bound; U-4 FX freshness window; U-5 Stake rounding semantics; U-6 coverage target; U-7 API P95/P99 latency; U-8 read-model rebuild time. None guessed; each is a clarification item or a risk above.

**Assessment boundary:** final PASS/CONCERNS/FAIL belongs to `nfr-assess` once implementation evidence exists.

---

### Testability Concerns and Architectural Gaps

#### 1. Blockers to Fast Feedback

| Concern | Impact | What Architecture Must Provide | Owner | Timeline |
|---|---|---|---|---|
| **No wallet test-signer seam** | No automated E2E on any money-moving journey; manual testing only | A test-only signer satisfying the Wallets Kit interface, injectable by build flag and structurally impossible to enable in production | Founder/Dev | Before Story 1.2 |
| **No chain-state seeding harness** | Mid-Cycle states unreachable for API/PWA tests; every UI state test needs a full manual setup | Scripted invocation sequences with ledger-time control producing named fixture states | Founder/Dev | Before Story 1.2 |
| **KYC provider unnamed** | Stories 1.6 and 1.8 unbuildable to a real contract; stubs encode guesses | A named provider, or a frozen adapter interface with guesses recorded as explicit assumptions | Founder | Before Story 1.6 |
| **Stake rounding unspecified** | Tests cannot assert values the specification does not determine | Rounding direction and operation order stated in the PRD | Founder (product) | Before Story 2.5 |

#### 2. Architectural Improvements Needed

1. **AD-15 conformance should be machine-checked**
   - **Current problem:** conformance is a design intention with no enforcement. A single entry point requiring detached auth falls outside the Wallets Kit module.
   - **Required change:** a CI check asserting every entry point authorizes only the transaction source, with no detached auth entry and no multi-party authorization.
   - **Impact if not fixed:** discovered late, the primary mobile surface requires a bespoke WalletConnect integration inside an eight-week window.
   - **Owner:** Founder/Dev — **Timeline:** before Story 1.7

2. **Stalled-Room TTL rescue should be decided, not deferred past the build**
   - **Current problem:** the spine's Deferred list acknowledges no cheap rescue path for a Room frozen long enough to approach archival, and proposes permissionless `extend_room_ttl` as the likely fix. In-test archival behaviour does not faithfully reproduce on-network behaviour.
   - **Required change:** decide whether `extend_room_ttl` lands in V1; schedule a testnet soak either way.
   - **Impact if not fixed:** a stalled pilot Room could become permanently unusable with member funds inside it.
   - **Owner:** Founder/Dev — **Timeline:** before the mainnet pilot

3. **Observability for off-chain services is entirely unspecified**
   - **Current problem:** no logging, metrics, tracing, or correlation-ID requirements for indexer, API, trust, or rates. Hosting is deferred.
   - **Required change:** minimally, structured logs with a correlation identifier on the indexer's event-processing path.
   - **Impact if not fixed:** read-model drift is diagnosed by archaeology rather than by evidence.
   - **Owner:** Founder/Dev — **Timeline:** before the mainnet pilot (low urgency at pilot scale)

---

### Testability Assessment Summary

#### What Works Well

This architecture is markedly more testable than is typical for its domain, and several decisions are effectively testability decisions:

- **AD-11 — no randomness anywhere.** Every outcome is a pure function of committed state and ledger timestamp. Ordering, Slash amounts, and score changes are exactly assertable, making property-based and golden-file testing viable rather than aspirational. This is the single largest testability asset in the system.
- **AD-4 — ledger timestamp compared against deadlines held in state.** Time is an input, not an ambient. Every time-dependent branch is deterministically reachable with no sleeps and no clock mocking, which removes the flakiness class that usually dominates scheduled-system suites.
- **Errors convention — one `#[contracterror]` enum per contract, stable discriminants, never renumbered.** Tests assert on codes rather than strings, so negative-path assertions survive refactoring.
- **Events convention — every value-moving transition emits an event; the indexer is built only from events, never state polling.** Read models are replayable by construction, making replay-equality a cheap, high-value test.
- **AD-2 — per-Room contract instances.** Test isolation is structural; parallel safety comes free at the contract level.
- **AD-8 — generated bindings, hand-written encoding prohibited.** Client/contract type drift becomes a build failure rather than a runtime money bug.
- **AD-13 — permissionless, idempotent `advance_round`.** Trivially testable from any address, and it removes the keeper from the liveness path.

#### Accepted Trade-offs (No Action Required)

For the V1 pilot the following are acceptable:

- **Single operator key for Registry writes (R-019).** A single-operator, single-Underwriter pilot with a known cohort. Revisit before any third-party Underwriter onboards, as the spine already states.
- **Fairness Floor untested in production (R-018).** Structurally unexercisable with one Room and one Cycle. Simulated coverage is the correct substitute; real evidence arrives with cohort 2.
- **No hosting, API latency, or DR posture (U-7, U-8).** Deferred deliberately in the spine; a cohort under twelve makes hosting a non-constraint. This is technical debt to revisit before the second cohort, not a pilot blocker.
- **No billing implementation.** The PRD states V1 tests no monetization; the subscription thesis is knowingly untested.

---

### Risk Mitigation Plans (High-Priority Risks ≥6)

Detailed strategies for the critical risk and the eleven high-priority risks. These must be addressed before the mainnet pilot; the testability blockers (R-010, R-011, R-012) must be addressed before Story 1.2.

#### R-001: Backstop adequacy insufficient (Score 9) — CRITICAL, GATE BLOCKER

**Mitigation Strategy:**

1. Build the full exposure table: for a 10-Member Room, enumerate every default pattern — which Members default, in which Rounds, relative to their own Payout Position.
2. For each pattern compute the maximum Backstop draw, with particular attention to the Round-1 recipient defaulting immediately after Payout (~9 Contributions outstanding against a Stake covering ~2).
3. Identify every pattern where total draw exceeds the posted Backstop.
4. Correct the formula, or constrain Room parameters (Member count, Contribution) so no pattern produces a shortfall.
5. Re-derive and publish the corrected minimum, updating FR-19 and the assumptions index.

**Owner:** Founder (product) · **Timeline:** before posting Backstop as pilot Underwriter; before Story 5.5 · **Status:** Planned
**Verification:** parameterized simulation over the full pattern space asserting zero recipient shortfall in every branch. Until this passes, the mainnet gate stays closed. This is a correction, not a waiver candidate.

#### R-002: Privileged path to escrow (Score 6)

**Mitigation Strategy:**

1. Enumerate every entry point across Registry, Factory, and Room.
2. Enumerate caller classes: Member, non-member, Underwriter, Registry admin, Factory, stranger.
3. Assert the full negative matrix — every combination that must revert, reverts with the expected error code.
4. Assert positively that escrow leaves the contract on exactly three code paths: Payout, Slash, cancellation refund.
5. Confirm no upgrade mechanism can alter an in-flight Room (AD-6).

**Owner:** Founder/Dev · **Timeline:** before testnet · **Status:** Planned
**Verification:** negative-authorization suite green; audit-readiness review confirms no reachable privileged path.

#### R-003: Waterfall not atomic (Score 6)

**Mitigation Strategy:**

1. Enumerate the waterfall legs: Slash, Backstop draw, Payout, score update, Round advance.
2. Inject failure at each leg and assert the entire invocation reverts with no partial state.
3. Assert the recipient holds the full Pot before any Round advance is observable.
4. Cover every branch: no default; cured in Grace; Stake covers; Stake partial plus Backstop; Stake exhausted with removal.

**Owner:** Founder/Dev · **Timeline:** before testnet · **Status:** Planned
**Verification:** fault-injection suite green across all legs and all branches.

#### R-004: Trust Score not independently reproducible (Score 6)

**Mitigation Strategy:**

1. Build the Story 4.5 reproduction harness sharing no code with `services/trust`.
2. Replay recorded event histories through both and assert exact equality.
3. Hunt the specific non-determinism sources: map/set iteration order, ambient clock, floating point, unstable sort.
4. Record the score function version with every Registry commit, and assert version-tagged replay stability.

**Owner:** Founder/Dev · **Timeline:** before Epic 4 close · **Status:** Planned
**Verification:** independent reproduction matches on every recorded history, including histories containing Defaults and removals.

#### R-005: Personal data reaches the chain (Score 6)

**Mitigation Strategy:**

1. Define a field-level allowlist for everything the KYC adapter may write to Registry.
2. Assert contract-side that Registry stores only wallet address, opaque provider reference, tier, and integers.
3. Assert the provider reference is opaque — not a hash or derivation of any identity document.
4. Gate any Registry schema change behind explicit review.

**Owner:** Founder/Dev · **Timeline:** before testnet · **Status:** Planned
**Verification:** allowlist tests green; schema assertion in the contract suite; confirmed in the audit-readiness review.

#### R-006: Soroban archival renders a Room unusable (Score 6)

**Mitigation Strategy:**

1. Assert TTL extension on every state-mutating invocation across all value-bearing entries.
2. Assert no contract logic branches on entry expiry (AD-4).
3. Decide whether permissionless `extend_room_ttl` lands in V1.
4. Run a stalled-Room soak on testnet — in-`Env` archival behaviour is not a faithful proxy.

**Owner:** Founder/Dev · **Timeline:** before mainnet pilot · **Status:** Planned
**Verification:** TTL assertions green; testnet soak shows a stalled Room remains recoverable.

#### R-007: AD-15 violated by an entry point (Score 6)

**Mitigation Strategy:**

1. Assert every entry point carries exactly one `require_auth` on the transaction source.
2. Add a CI check that fails on any detached-auth or multi-party-auth requirement.
3. Exercise the real path end to end through Wallets Kit's WalletConnect module by Story 1.7.

**Owner:** Founder/Dev · **Timeline:** before Story 1.7 · **Status:** Planned
**Verification:** CI check present and failing correctly on a deliberately introduced violation.

#### R-008: Reputation-over-collateral thesis falsified (Score 6)

**Mitigation Strategy:**

1. Record, for every Default, the defaulter's Trust Score and Payout Position.
2. Surface the distribution so concentration among high-trust early recipients is visible in pilot data.
3. Predefine what would count as falsification, before the pilot produces data.

**Owner:** Founder (product) · **Timeline:** pilot measurement · **Status:** Planned
**Verification:** the pilot retrospective can answer the concentration question from recorded data rather than argument. Note this risk is not mitigable by testing — the Backstop absorbs the cost either way; the mitigation is ensuring the pilot produces an answer.

#### R-009: Stake rounding unspecified (Score 6)

**Mitigation Strategy:**

1. Specify rounding direction and operation order in FR-13 — integer arithmetic throughout, no floating point.
2. Prefer rounding that never produces a Stake below the intended collateral.
3. Test boundaries across the full 0–1000 score domain, including 0, 1000, and values where the division does not divide evenly.

**Owner:** Founder (product) · **Timeline:** before Story 2.5 · **Status:** Planned
**Verification:** boundary suite asserts exact expected values from a now-determinate specification.

#### R-010: KYC provider unnamed (Score 6)

**Mitigation Strategy:**

1. Name the provider, or freeze an explicit adapter interface covering webhook payload, duplicate-detection signal, tier vocabulary, retry and idempotency semantics.
2. Build the stub to that interface, recording every guess as an explicit assumption.
3. Assert webhook idempotency — the same event delivered twice issues exactly one attestation.

**Owner:** Founder · **Timeline:** before Story 1.6 · **Status:** Planned
**Verification:** adapter interface frozen and documented; stories 1.6 and 1.8 buildable without further invention.

#### R-011: No wallet test-signer seam (Score 6)

**Mitigation Strategy:**

1. Define a test-only signer satisfying the Wallets Kit interface, signing with a known keypair.
2. Inject by build flag, structurally impossible to enable in production builds.
3. Assert in CI that production bundles cannot contain it.

**Owner:** Founder/Dev · **Timeline:** before Story 1.2 · **Status:** Planned
**Verification:** an E2E journey completes join and contribute unattended; production bundle assertion green.

#### R-012: No chain-state seeding harness (Score 6)

**Mitigation Strategy:**

1. Build the seeding harness alongside the test framework in Story 1.2.
2. Script named fixture states: filling, started, grace-open, default-resolved, member-removed, closed.
3. Include ledger-time control so time-dependent states are reachable on demand.

**Owner:** Founder/Dev · **Timeline:** before Story 1.2 · **Status:** Planned
**Verification:** every named fixture state reachable by one command against a local network.

---

### Assumptions and Dependencies

#### Assumptions

1. The eleven `[ASSUMPTION]` values in PRD §14 (score deltas, starting scores, Stake formula, Backstop minimum, open window, capacity, weekly-only Cadence, tiebreak rule) are treated as specification for test purposes. Where the pilot may falsify one — notably the Backstop minimum (R-001) and the score deltas — tests assert current behaviour, and changing the value is a specification change requiring test updates.
2. Rust contract tests under `soroban_sdk::Env` faithfully reproduce on-network behaviour for everything except state archival and real transaction cost, which require testnet evidence.
3. A local Stellar quickstart network is available for PR-time E2E; testnet is reserved for Epic 5 acceptance runs.
4. The architecture's Deferred items (upgradeability, hosting topology, fee accrual mechanics, multi-Room trust effects) stay deferred and do not acquire test coverage in V1.
5. `stellar-cli`, `@stellar/stellar-sdk`, Stellar Wallets Kit, React and Vite versions are pinned at install; version drift is a CI concern once pinned.

#### Dependencies

1. **KYC provider named or adapter interface frozen** — required before Story 1.6
2. **Test-signer seam defined** — required before Story 1.2
3. **Chain-state seeding harness** — required before Story 1.2
4. **Stake rounding specified** — required before Story 2.5
5. **Transaction-cost threshold ratified (U-1)** — required before Story 5.4
6. **FX provider selected and freshness window ratified (U-4)** — required before Story 2.8
7. **Corrected Backstop formula (R-001)** — required before Story 5.5 and before the mainnet gate
8. **PH fintech counsel (milestone T3)** — required before mainnet, outside the build window

#### Risks to Plan

- **Risk:** Test infrastructure (signer seam, seeding harness) is itself a substantial build — roughly 30–50 hours — landing in Story 1.2 at the very start of an eight-week window.
  - **Impact:** slipping it delays every subsequent story's testability, and tests get written after the fact or not at all.
  - **Contingency:** contract tests need none of this infrastructure and can proceed in parallel. If the harness slips, protect contract-level P0 coverage first — it carries every money-safety invariant — and let UI E2E lag.
- **Risk:** No dedicated QA engineer; test development competes with feature development for the same person.
  - **Impact:** under schedule pressure, the P0 invariant suite is the natural casualty, and it is the one thing that cannot be safely cut.
  - **Contingency:** treat the P0 set as story acceptance criteria rather than as a separate test task, so it cannot be descoped independently of the feature.
- **Risk:** R-001 remains open at the mainnet gate.
  - **Impact:** the gate stays closed, delaying the pilot.
  - **Contingency:** the exposure table is analysis, not implementation — it can be produced in parallel with the build and does not depend on any code landing.

---

**End of Architecture Document**

**Next Steps for Architecture:**

1. Resolve the five blockers in the Quick Guide — items 2, 3, and 4 gate Story 1.2, which is second in the sprint plan
2. Confirm owners and timelines for the twelve risks at score ≥6
3. Ratify or reject the eight UNKNOWN thresholds
4. Validate assumptions and dependencies above

**Next Steps for Test Development:**

1. Contract-level P0 coverage needs none of the blocked infrastructure — start there
2. See `test-design-qa.md` for scenario tables and execution strategy
3. Build the test-signer seam and seeding harness inside Story 1.2
