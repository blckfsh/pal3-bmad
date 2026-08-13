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
---

# Test Design for QA: Pal3 V1

**Purpose:** Test execution recipe. Defines what to test, at which level, and what test development needs from elsewhere before it can start.

**Date:** 2026-08-13
**Author:** Murat (TEA Master Test Architect)
**Status:** Draft
**Project:** paluwagan3

**Related:** See `test-design-architecture.md` for testability concerns, risk mitigation plans, and architectural blockers. Risk IDs (R-001…R-021) and threshold IDs (U-1…U-8) are shared between both documents.

---

## Executive Summary

**Scope:** All 5 epics and 45 stories of the Pal3 V1 pilot — Soroban contracts (Registry, Factory, Room), off-chain services (trust, KYC, indexer, rates, API), and the member/underwriter PWA.

**Risk Summary:**

- Total risks: 21 — 1 critical (score 9), 11 high (score ≥6), 9 medium, 0 low
- Critical categories: SEC and TECH carry the most high-priority risks; the single critical risk (R-001, Backstop adequacy) is BUS

**Coverage Summary:**

- P0: ~22 scenarios (money-safety invariants, authorization negatives, determinism)
- P1: ~35 scenarios (mechanics, read models, key screens)
- P2: ~32 scenarios (edge cases, secondary surfaces, UI states)
- P3: ~10 scenarios (benchmarks, exploratory, documentation)
- Supporting infrastructure: ~25 items (signer seam, seeding harness, factories, CI)
- **Total: ~124 scenarios (~4.5–7 weeks, one engineer full-time)**

**Stack note that shapes everything below.** The dominant test surface is the **Rust contract suite running under `soroban_sdk::Env`**, not an HTTP API. That is where authoritative state lives (AD-1) and therefore where every money-moving invariant is proven. Playwright applies to the PWA and to a deliberately thin set of cross-surface journeys. `playwright-utils` is enabled in config and used at that layer; it has no role in contract testing.

---

## Not in Scope

| Item | Reasoning | Mitigation |
|---|---|---|
| **KYC provider's internal duplicate detection** | Out of Pal3's technical boundary by design (PRD §4.1); the provider is the sole controller of document data | Test Pal3's *enforcement* of the provider's result: a duplicate-flagged attestation must be rejected at issue and must revert on join |
| **Identity document handling** | Never touches Pal3 systems (AD-10) | Assert the negative — no personal data reaches Registry (P0-011) |
| **Billing / subscription payment** | V1 implements no billing (PRD §11) | Capacity gating is tested; payment is not |
| **Multi-Room and multi-Underwriter behaviour** | Structurally absent in V1 — one Room, one Underwriter | Deferred to cohort 2 along with the corresponding open questions |
| **Trust Score decay and rehabilitation** | Deferred past V1; no decay horizon exists within a single Cycle | Deferred |
| **Peso rail / on-ramp** | Out of MVP scope (PRD §6.2) | FX is presentational only and tested as such |
| **API load and latency at scale** | Cohort under twelve; hosting explicitly deferred in the spine | U-7 recorded as an unknown threshold, revisit before cohort 2 |
| **Upgradeability mechanism** | Deferred to the audit-readiness pass | AD-6's constraint (no upgrade may alter an in-flight Room) is asserted; the mechanism is not |
| **Regulatory posture** | Requires PH fintech counsel (milestone T3) | Manual sign-off, not automatable |

---

## Dependencies & Test Blockers

**Test development cannot proceed on the affected surfaces without these.**

### Architecture Dependencies (Pre-Implementation)

**Source:** see the Quick Guide in `test-design-architecture.md` for full mitigation plans.

1. **Wallet test-signer seam** — Founder/Dev — before Story 1.2 (R-011)
   - Needed: a test-only signer satisfying the Stellar Wallets Kit interface, signing with a known keypair, injectable by build flag.
   - Blocks: every automated E2E journey that moves money. WalletConnect's pairing flow is human-in-the-loop; without a seam there is no unattended path through join, contribute, or payout.

2. **Chain-state seeding harness** — Founder/Dev — before Story 1.2 (R-012)
   - Needed: scripted invocation sequences plus ledger-time control producing named fixture states — filling, started, grace-open, default-resolved, member-removed, closed.
   - Blocks: all API and PWA testing of mid-Cycle states. Free inside `Env`; absent against a deployed network.

3. **KYC adapter interface frozen** — Founder — before Story 1.6 (R-010)
   - Needed: webhook payload shape, duplicate-detection signal, tier vocabulary, retry/idempotency semantics.
   - Blocks: Stories 1.6 and 1.8. A stub built before this encodes guesses the real provider will contradict.

4. **Stake rounding specified** — Founder (product) — before Story 2.5 (R-009, U-5)
   - Needed: rounding direction and operation order for `Contribution × (2 − TrustScore/1000)` in integer stroops.
   - Blocks: any assertion of an expected Stake value. A test cannot assert what the specification does not determine.

5. **Transaction-cost threshold ratified** — Founder — before Story 5.4 (U-1)
   - Needed: a maximum cost per Round, as a percentage of Contribution or an absolute ceiling.
   - Blocks: Story 5.4's pass/fail criterion. Measurement without a target produces a number, not a verdict.

6. **Corrected Backstop formula** — Founder (product) — before Story 5.5 and the mainnet gate (R-001)
   - Needed: a minimum that survives every default pattern, including the Round-1 recipient defaulting immediately after Payout.
   - Blocks: Story 5.5's target. The exposure simulation can be built now; what it asserts against cannot.

### Test Infrastructure Setup (Pre-Implementation, Story 1.2)

1. **Contract test harness** — Rust
   - `Env`-based fixtures for Registry, Factory, and a deployed Room instance
   - Ledger-time helpers so Round boundaries and Grace Windows are reachable by construction
   - Builders for Member cohorts at specified Trust Scores, and for Rooms at specified lifecycle states
   - Error-code assertion helpers keyed to each contract's `#[contracterror]` enum

2. **Service test setup** — TypeScript / Vitest
   - Event-history factories for indexer replay and score reproduction
   - KYC webhook stub built to the frozen adapter interface
   - FX rate stub covering fresh, stale, and unavailable

3. **Test environments**
   - Local: Stellar quickstart network for PR-time E2E
   - CI: `cargo test` plus Vitest plus Playwright against local quickstart
   - Testnet: reserved for Epic 5 acceptance — cost measurement, invariant verification, stalled-Room soak

**Example E2E pattern** (`tea_use_playwright_utils` is enabled; this is the PWA layer only):

```typescript
import { test } from '@seontechnologies/playwright-utils/api-request/fixtures';
import { expect } from '@playwright/test';

test('@P1 @API room read model reflects a seeded grace-open state', async ({ apiRequest }) => {
  const { status, body } = await apiRequest({
    method: 'GET',
    path: `/api/rooms/${process.env.SEEDED_ROOM_ADDRESS}/current-round`,
  });

  expect(status).toBe(200);
  expect(body.state).toBe('grace_open');
  expect(body.graceDeadline).toBeDefined();
  expect(body.members.filter((m) => m.status === 'in_grace')).toHaveLength(1);
});
```

Contract tests use no such framework — they are ordinary `#[test]` functions against `Env`, which is what makes them fast enough to run in full on every PR.

---

## Risk Assessment

**Note:** full risk details, mitigations, owners, and timelines are in `test-design-architecture.md`. This summarizes how each risk is validated.

### Critical and High-Priority Risks (Score ≥6)

| Risk ID | Category | Description | Score | Test Coverage |
|---|---|---|---|---|
| **R-001** | BUS | Backstop insufficient against its own worst case | **9** | P0-007 exposure-table simulation across every default pattern, asserting zero recipient shortfall |
| **R-002** | SEC | Privileged path to escrow | **6** | P0-003/004/005/006 — exhaustive negative-authorization matrix and escrow exit-path audit |
| **R-003** | TECH | Waterfall not atomic | **6** | P0-001/002 — full-Pot assertion in every branch plus fault injection on every leg |
| **R-004** | DATA | Trust Score not independently reproducible | **6** | P0-008/009 — independent reproduction harness and ordering determinism |
| **R-005** | SEC | Personal data reaches the chain | **6** | P0-011 — Registry schema assertion and KYC adapter field allowlist |
| **R-006** | TECH | Soroban archival renders a Room unusable | **6** | P0-018/019 in `Env`; weekly testnet stalled-Room soak for on-network truth |
| **R-007** | TECH | AD-15 violated by an entry point | **6** | P0-017 — single `require_auth` assertion plus a static CI check |
| **R-008** | BUS | Reputation-over-collateral thesis falsified | **6** | Not test-mitigable. P1 instrumentation records Trust Score and Payout Position on every Default so the pilot can answer it |
| **R-009** | DATA | Stake rounding unspecified | **6** | P0-010 — boundary assertions across the full 0–1000 score domain, once specified |
| **R-010** | OPS | KYC provider unnamed | **6** | Blocks P1 webhook coverage until the interface is frozen; idempotency test written against the frozen interface |
| **R-011** | OPS | No wallet test-signer seam | **6** | Blocks the entire E2E journey set; contract P0 coverage is unaffected |
| **R-012** | OPS | No chain-state seeding harness | **6** | Blocks API and PWA state coverage; contract P0 coverage is unaffected |

### Medium-Priority Risks (Score 3–5)

| Risk ID | Category | Description | Score | Test Coverage |
|---|---|---|---|---|
| R-013 | BUS | Grace Window abuse, no escalation | 4 | P1 — repeated-cure sequence simulation; asserts current (unescalated) behaviour until the rule changes |
| R-014 | TECH | AD-5 snapshot guard missing | 4 | P0-015 — join carrying a stale score must revert |
| R-015 | OPS | Off-chain outage claim unverified | 4 | Weekly outage drill — services down, complete a Round by direct invocation |
| R-016 | PERF | Per-Round cost exceeds constraint | 4 | Nightly measured-cost run on the worst-case waterfall invocation, not the happy path (U-1 blocks the verdict) |
| R-017 | DATA | Indexer read-model drift | 4 | P1 — replay-equality tests; events-only construction assertion |
| R-018 | TECH | Fairness Floor unexercisable in V1 | 3 | P2 — simulated multi-Cycle unit tests constructing the precondition synthetically |
| R-019 | SEC | Trust service operator key compromise | 3 | Accepted for the pilot; no test coverage. Revisit before third-party Underwriters |
| R-020 | TECH | FX provider and freshness window undefined | 3 | P1 — rate-omission path tested explicitly; threshold value blocked on U-4 |
| R-021 | SEC | External token-contract call inside the atomic waterfall | 3 | P1 — state consistency asserted across the token call boundary; audit review carries the rest |

---

## NFR Test Coverage Plan

| NFR Category | Requirement / Threshold | Planned Validation | Tool / Level | Evidence Artifact | Priority |
|---|---|---|---|---|---|
| Security — funds safety | No privileged path to escrow, Positions, or Slash reversal (binary) | Exhaustive negative-authorization matrix; escrow exit-path audit | Contract (Rust/`Env`) | `cargo test` report; audit-readiness review (Story 5.7) | P0 |
| Security — privacy | No personal data on-chain (binary) | Registry schema assertion; KYC adapter field allowlist | Contract + Service | Test report; adapter allowlist test | P0 |
| Reliability — atomicity | Full success or full revert; recipient always holds the full Pot | Fault injection on every waterfall leg; branch matrix | Contract | Fault-injection suite report | P0 |
| Reliability — determinism | Reproducible from event history by any observer | Independent reproduction harness; ordering determinism across builds | Service + Contract | Story 4.5 reproduction output | P0 |
| Reliability — durability | Persistent storage, TTL extended on every write; expiry never control flow | TTL assertions in `Env`; stalled-Room soak on testnet | Contract + on-chain | Test report; testnet soak result | P0 / P2 |
| Reliability — liveness | In-flight Rooms operable during total off-chain outage | Outage drill: services down, complete a Round by direct invocation | On-chain | Drill report | P1 |
| Performance — cost | Cost per Round negligible vs a ₱1,000-equivalent Contribution — **threshold UNKNOWN (U-1)** | Measure worst-case waterfall invocation on testnet | On-chain | Story 5.4 measured cost report | P1 |
| Compliance | DPA minimization; SEC/BSP/IC posture | Automated: on-chain data assertions. Manual: counsel review (T3) | Contract + manual | Test report; counsel sign-off | P0 / manual |
| Maintainability | Bindings generated not hand-written; one handle implementation | CI regeneration-drift check; single-module assertion | CI | Zero-diff check; coverage report | P1 |
| Availability — API | Read-model staleness bound — **UNKNOWN (U-3)** | Replay-equality; freshness surfaced once bounded | Service | Replay test report | P1 |

**Missing thresholds or evidence sources:** U-1 (cost ceiling), U-2 (Round-boundary availability target — may be N/A given AD-13's permissionless advance; confirm), U-3 (read-model staleness bound), U-4 (FX freshness window), U-5 (Stake rounding), U-6 (coverage target), U-7 (API latency), U-8 (read-model rebuild time). Each needs stakeholder ratification before `nfr-assess` can render a verdict on the affected category.

---

## Entry Criteria

- [ ] The five architecture blockers resolved (test-signer seam, seeding harness, KYC interface, Stake rounding, cost threshold)
- [ ] Contract test harness available with `Env` fixtures and ledger-time helpers (Story 1.2)
- [ ] Local Stellar quickstart network running in CI
- [ ] Named fixture states reachable by one command
- [ ] Error-code enums defined per contract with stable discriminants
- [ ] PRD assumptions confirmed as specification, or amended

Contract-level P0 work depends on none of the blocked items and can begin as soon as the harness exists.

## Exit Criteria

- [ ] All P0 tests passing — 100%, no waivers
- [ ] P1 tests ≥95% passing, every failure triaged and explicitly accepted
- [ ] No open high-severity defects
- [ ] Every risk at score ≥6 mitigated or formally waived with owner, reason, and expiry
- [ ] R-001 **closed, not waived** — the exposure simulation shows zero shortfall in every default pattern
- [ ] Contract branch coverage ≥90% on `contracts/room`, ≥80% elsewhere `[PROPOSED — U-6]`
- [ ] Binding regeneration produces zero diff
- [ ] Stories 5.3, 5.4, 5.5 complete and passing before the mainnet gate
- [ ] Audit-readiness review passed (Story 5.7, milestone T2)

---

## Test Coverage Plan

**IMPORTANT:** P0/P1/P2/P3 indicate **priority and risk level** — what to protect first if time runs short — not execution timing. See Execution Strategy for when things run.

### P0 (Critical)

**Criteria:** Blocks core functionality + high risk (≥6) + no workaround.

| Test ID | Requirement | Test Level | Risk Link | Notes |
|---|---|---|---|---|
| **P0-001** | Recipient receives the full Pot in every default branch — none, cured, Stake covers, Stake partial + Backstop, Stake exhausted + removal | Contract | R-001, R-003 | The product's central promise (SM-3). Five branches, all asserted against the exact Pot |
| **P0-002** | Waterfall atomicity — injected failure on any leg reverts the whole invocation with no partial state | Contract | R-003 | Legs: Slash, Backstop draw, Payout, score update, Round advance |
| **P0-003** | Negative-authorization matrix — every entry point × Member, non-member, Underwriter, Registry admin, Factory, stranger | Contract | R-002 | Assert expected error code, not just revert |
| **P0-004** | No caller can alter a Payout Position after Room start | Contract | R-002 | AD-6 |
| **P0-005** | No caller can reverse a Slash | Contract | R-002 | AD-6 |
| **P0-006** | Escrow leaves the contract on exactly three paths — Payout, Slash, cancellation refund — and no other | Contract | R-002 | Exit-path audit; assert balance conservation |
| **P0-007** | Backstop exposure table — parameterized sweep over every default pattern in a 10-Member Room asserting zero recipient shortfall | Contract (table/property) | R-001 | The gate-blocking test. Full sweep runs nightly; a representative subset runs per PR |
| **P0-008** | Trust Score independently reproduced from event history by a harness sharing no code with `services/trust` | Service | R-004 | Story 4.5. Sharing code proves only self-agreement |
| **P0-009** | Payout ordering is deterministic — same snapshots yield the same order across repeated runs and separate builds, including the tiebreak | Contract | R-004 | AD-11. Hunts map-iteration and unstable-sort non-determinism |
| **P0-010** | Stake sizing exact across the full 0–1000 score domain, including rounding boundaries and non-dividing values | Contract | R-009 | Blocked on U-5 being specified |
| **P0-011** | Registry stores only wallet address, opaque provider reference, tier, integers — and the reference is not derived from any identity document | Contract + Service | R-005 | Irreversible if wrong (AD-10, DPA) |
| **P0-012** | Duplicate-flagged attestation rejected at issue; join with a duplicate-flagged attestation reverts | Contract | FR-3 | |
| **P0-013** | Join without a valid KYC attestation reverts | Contract | FR-1 | |
| **P0-014** | `advance_round` callable by any address, idempotent, identical effect regardless of caller, reverts if called early | Contract | AD-13 | Liveness must not depend on a keeper |
| **P0-015** | Join reverts when the Registry score no longer matches the score carried in the transaction | Contract | R-014 | AD-5 — protects the terms a Member agreed to |
| **P0-016** | Room parameters immutable after start — amount, Cadence, membership, ordering | Contract | FR-6 | Removal via Stake exhaustion is the only permitted membership change |
| **P0-017** | Every entry point authorizes only the transaction source via a single `require_auth`; no detached auth, no multi-party auth | Contract + CI | R-007 | AD-15. Static check fails the build on violation |
| **P0-018** | Value-bearing state uses persistent storage; TTL extended on every state-mutating invocation; `temporary` never used for fund-affecting data | Contract | R-006 | AD-3 |
| **P0-019** | No contract logic branches on entry expiry; all time transitions compare ledger timestamp to a stored deadline | Contract | R-006 | AD-4 |
| **P0-020** | Cancellation after the open window returns every Stake and Contribution in full | Contract | FR-6 | Balance conservation asserted |
| **P0-021** | Contributions accepted only for the current open Round; double-payment rejected | Contract | FR-8 | |
| **P0-022** | Room close returns full Stakes to every completing Member and settles the Underwriter | Contract | FR-7 | Balance conservation asserted |

**Total P0:** ~22 scenarios

> P0 runs at ~18% of total rather than the usual <10%. This is deliberate: the set is almost entirely invariants rather than features. In a system custodying members' money with no privileged override and no recovery mechanism, an invariant demoted to P1 is an invariant that can regress silently into mainnet.

---

### P1 (High)

**Criteria:** Important features + medium/high risk + common workflows.

| Test ID | Requirement | Test Level | Risk Link | Notes |
|---|---|---|---|---|
| **P1-001** | Grace Window opens on a missed deadline; no Round advance and no Payout while open | Contract | R-013 | FR-14 |
| **P1-002** | Contribution inside the Grace Window accepted in full and recorded as cured (+2, not −150) | Contract | R-013 | |
| **P1-003** | Repeated-cure sequence across a full Cycle — asserts current unescalated behaviour | Contract | R-013 | Documents the open question rather than hiding it |
| **P1-004** | Grace notifications fire at window open and at 24 hours remaining | Service | — | FR-14 |
| **P1-005** | Removal occurs at the moment a Slash leaves the Stake at zero | Contract | R-001 | FR-16 |
| **P1-006** | Backstop covers a removed Member's Contributions for all remaining Rounds | Contract | R-001 | The worst-case exposure path |
| **P1-007** | Removed Member who has not been paid receives nothing; one already paid retains it and the Backstop absorbs the loss | Contract | R-001 | |
| **P1-008** | Abandonment penalty (−400) applied on removal | Contract | — | FR-10/FR-16 |
| **P1-009** | Underwriter fee charged on Contribution, never on Payout; the Pot is never reduced | Contract | — | FR-20 |
| **P1-010** | Platform takes no share of the fee and no share of the Pot | Contract | — | FR-20 |
| **P1-011** | Room creation above tier capacity reverts; non-weekly Cadence reverts | Contract | — | FR-4/FR-18 |
| **P1-012** | Room cannot admit Members until the required Backstop is posted and locked | Contract | — | FR-19, AD-14 |
| **P1-013** | Backstop cannot be withdrawn while the Room it collateralizes is in progress | Contract | — | FR-19 |
| **P1-014** | Undrawn Backstop returns to the Underwriter at Room close | Contract | — | AD-14 |
| **P1-015** | Maximum exposure computable before the Underwriter commits; live exposure updates on every Slash and draw | Contract + Service | R-008 | FR-17 |
| **P1-016** | Every Default records the defaulter's Trust Score and Payout Position | Service | R-008 | Instrumentation for the pilot's designated falsification target |
| **P1-017** | Trust Score clamped to [0, 1000] after every update; capped at 400 until one Cycle completes | Service + Contract | R-004 | FR-2/FR-10 |
| **P1-018** | All five score deltas applied on their defined events | Service | R-004 | FR-10 |
| **P1-019** | Trust and KYC services write to Registry only between Rooms, never during an active Cycle | Service | — | AD-7 |
| **P1-020** | Read-model replay equality — the same event sequence yields identical models across runs | Service | R-017 | AD-1 rebuildability |
| **P1-021** | Indexer builds read models only from events, never from state polling | Service | R-017 | |
| **P1-022** | API omits the rate entirely outside the freshness window; the client renders no peso line when no rate is served | Service + Component | R-020 | AD-16. Threshold blocked on U-4; the omission path is testable now |
| **P1-023** | Every peso figure carries its rate source identifier and observation timestamp | Component | R-020 | |
| **P1-024** | No rate value ever reaches a contract; rate-adapter unavailability never blocks join, Contribution, or Payout | Contract + Service | R-020 | AD-16 |
| **P1-025** | Binding regeneration produces zero diff; generated clients compile against services and app | CI | — | AD-8 |
| **P1-026** | Handle derivation is deterministic and identical between client and indexer — one module, one result | Service + Component | — | AD-17 |
| **P1-027** | Terms screen shows all required fields — amount, Cadence, Member count, Payout Position and date, Stake, fee, waterfall in plain language | Component | — | FR-22 |
| **P1-028** | FX disclosure acknowledged before joining and the acknowledgement recorded | Component + Service | — | FR-22 |
| **P1-029** | Insufficient balance reports the exact shortfall before a join is attempted | Component | — | FR-21 |
| **P1-030** | Wallet connects over WalletConnect and a commit completes with a single signature | E2E | R-007, R-011 | Blocked on the signer seam |
| **P1-031** | Underwriter admission view exposes Trust Score, Cycles completed, Default count, account age — and nothing else | Component + Service | R-005 | FR-5 |
| **P1-032** | Rejected applicant's locked funds returned in full | Contract | — | FR-5 |
| **P1-033** | Payout Positions computed and frozen at Room start, visible to every Member beforehand | Contract + Component | — | FR-6/FR-11 |
| **P1-034** | Each Member receives exactly one Payout per Cycle, at the full Pot with no deduction | Contract | R-003 | FR-12 |
| **P1-035** | Cycle-completion credit (+50) applied at close for Members with zero Defaults | Contract | — | FR-7/FR-10 |

**Total P1:** ~35 scenarios

---

### P2 (Medium)

**Criteria:** Secondary features + low/medium risk + edge cases + regression prevention.

| Test ID | Requirement | Test Level | Risk Link | Notes |
|---|---|---|---|---|
| **P2-001** | Fairness Floor triggers correctly — simulated multi-Cycle precondition constructed synthetically | Contract | R-018 | Unexercisable in V1; simulation is the only available evidence |
| **P2-002** | Tiebreak by earlier Room-join timestamp, deterministic and auditable | Contract | R-004 | FR-11 |
| **P2-003** | Expired attestation blocks joining a new Room but does not remove a Member from one in progress | Contract | — | FR-1 |
| **P2-004** | Room filling and cancelled states render correctly through the full lifecycle | Component | — | Story 2.10 |
| **P2-005** | Grace banner — the highest-stakes state in the product — renders with correct time remaining and copy | Component | R-013 | Story 3.11 |
| **P2-006** | Room home shows current Round, my obligation, my payout date | Component | — | Story 3.8 |
| **P2-007** | Members status list shows paid / pending / in Grace / defaulted by handle only | Component | R-005 | FR-9 |
| **P2-008** | Round detail, default-resolved and removal states | Component | — | Story 3.12 |
| **P2-009** | Cycle summary at Room close | Component | — | Story 3.13 |
| **P2-010** | Underwriter room — live and maximum exposure display | Component | R-008 | Story 3.14 |
| **P2-011** | Trust surface — score, consequence, and every event that moved it | Component | — | Story 4.3; supports SM-5 legibility |
| **P2-012** | Next threshold and cycle carry-forward display | Component | — | Story 4.4 |
| **P2-013** | Account and Activity surfaces | Component | — | Story 1.9 |
| **P2-014** | Contract error codes map to user-facing copy; the client never parses messages | Component | — | Errors convention |
| **P2-015** | Design token foundation and PWA shell — install, offline shell, service worker | Component | — | Story 1.3 |
| **P2-016** | Testnet stalled-Room soak — a Room left inactive approaches archival and remains recoverable | On-chain | R-006 | Weekly; `Env` cannot prove this |

**Total P2:** ~32 scenarios (rows above group related cases)

---

### P3 (Low)

**Criteria:** Nice-to-have, exploratory, benchmarks, documentation validation.

| Test ID | Requirement | Test Level | Notes |
|---|---|---|---|
| **P3-001** | Transaction cost benchmark across all invocation types | On-chain | Story 5.4; verdict blocked on U-1 |
| **P3-002** | Contract source published under Apache 2.0 from first deployment | Manual | Story 5.1 |
| **P3-003** | Recorded testnet addresses match deployed contracts | On-chain | Story 5.2 |
| **P3-004** | Load behaviour beyond pilot scale — Rooms larger than 10 Members | Contract | Exploratory; SM-C3 resists this in production |
| **P3-005** | Accessibility pass on the primary member journey | Component | Mobile-first surface |
| **P3-006** | Exploratory session on the default waterfall from a member's perspective | Manual | Unscripted; the highest-consequence flow |

**Total P3:** ~10 scenarios

---

## Execution Strategy

**Philosophy:** run everything in PRs unless the infrastructure genuinely forbids it. Rust contract tests execute in seconds; Playwright parallelizes hundreds of tests into 10–15 minutes. There is no case for tiering functional tests by priority.

### Every PR (~10–15 min)

- Full `cargo test` contract suite — all P0/P1/P2 contract scenarios
- All TypeScript service tests (Vitest)
- Binding regeneration drift check
- UI component tests
- Thin E2E journey set against a local Stellar quickstart network
- A representative subset of the P0-007 exposure table

**Why in PRs:** fast feedback, no expensive infrastructure. The contract suite is the money-safety net and must never be deferred.

### Nightly (~30–60 min)

- Full P0-007 exposure-table sweep — every default pattern across a 10-Member Room, combinatorially too large for PR latency
- Testnet deployment smoke
- Measured transaction-cost run on the worst-case waterfall invocation (U-1 evidence)
- Extended property-based runs on ordering and score reproduction

**Why deferred:** long-running parameter sweeps and testnet round-trips.

### Weekly (~hours)

- High-iteration invariant fuzzing
- Off-chain outage drill — indexer, API, trust and rate services down, complete a full Round by direct contract invocation (R-015, ASR-11)
- TTL/archival soak against a deliberately stalled Room (R-006)

**Why deferred:** very long-running; infrequent validation is sufficient.

### Manual (excluded from automation)

- Audit-readiness review (Story 5.7)
- Counsel confirmation of regulatory posture (milestone T3)
- Licence and publication verification (Story 5.1)
- Exploratory sessions

---

## Test Effort Estimate

| Priority | Count | Effort Range | Notes |
|---|---|---|---|
| P0 | ~22 | ~1.5–2.5 weeks | Invariant and negative-authorization suites, waterfall branch matrix, exposure simulation harness |
| P1 | ~35 | ~1.5–2 weeks | Grace/removal/fee/capacity mechanics, replay determinism, API and key screens |
| P2 | ~32 | ~3–6 days | Edge cases, UI state permutations, secondary surfaces |
| P3 | ~10 | ~1–2 days | Benchmarks, exploratory, documentation |
| Supporting infrastructure | ~25 | ~4–6 days | Signer seam, seeding harness, factories, CI wiring |
| **Total** | **~124** | **~4.5–7 weeks** | **One engineer, full-time** |

**Assumptions:**

- Includes test design, implementation, debugging, and CI integration
- Excludes ongoing maintenance (~10%)
- Assumes the five architecture blockers are resolved on schedule

**Framing.** There is no dedicated QA engineer. Most of this effort lands inside story implementation rather than a separate track — Story 1.2 owns the harness and CI, and each subsequent story carries its own tests. Against an eight-week build window this is substantial but proportionate for a funds-custody system, and it is why Story 1.2 sits second in Epic 1.

**Protecting the P0 set.** Because test work and feature work compete for the same person, the P0 invariant suite is the natural casualty under schedule pressure — and the one thing that cannot safely be cut. Recommendation: write the P0 scenarios into story acceptance criteria so they cannot be descoped independently of the feature they protect.

---

## Implementation Planning Handoff

| Work Item | Owner | Target Milestone | Dependencies / Notes |
|---|---|---|---|
| Contract test harness with `Env` fixtures and ledger-time helpers | Dev | Story 1.2 | None — start here |
| Wallet test-signer seam | Dev | Story 1.2 | Blocks all E2E (R-011) |
| Chain-state seeding harness with named fixture states | Dev | Story 1.2 | Blocks API/PWA state coverage (R-012) |
| CI pipeline: `cargo test`, Vitest, Playwright, binding-drift check, AD-15 static check | Dev | Story 1.2 | R-007 |
| Backstop exposure simulation | Dev | Story 5.5 | Buildable now; its target is blocked on R-001 |
| Independent score reproduction harness | Dev | Story 4.5 | Must share no code with `services/trust` (R-004) |
| KYC webhook stub | Dev | Story 1.6 | Blocked on the frozen adapter interface (R-010) |
| Negative-authorization matrix | Dev | Epic 2–3 | Grows as entry points are added |
| Outage drill runbook | Dev | Before mainnet | R-015 |

---

## Tooling & Access

| Tool or Service | Purpose | Access Required | Status |
|---|---|---|---|
| Stellar quickstart (local) | PR-time E2E and seeding | Local container | Pending — Story 1.2 |
| Stellar testnet | Epic 5 acceptance, cost measurement, soak | Funded testnet accounts | Pending |
| `stellar-cli` | `contract build`, `contract bindings` | Local install, version recorded at setup | Pending |
| KYC provider sandbox | Webhook integration | Provider account | **Blocked — provider unnamed (R-010)** |
| FX rate provider | Rate adapter | API key, likely free tier | **Blocked — provider unselected (R-020)** |
| CI runner | Test execution | `ci_platform` unset in config (`auto`) | Pending — decide in Story 1.2 |

**Access requests needed:**

- [ ] KYC provider sandbox credentials — after the provider is named
- [ ] FX rate provider key — before Story 2.8
- [ ] Funded testnet accounts for Underwriter and 10 Members

---

## Interworking & Regression

| Service / Component | Impact | Regression Scope | Validation |
|---|---|---|---|
| **Registry** | Written by trust and KYC services; read by every Room | Attestation and score tests; cross-Room isolation | A Registry change must not alter any in-flight Room's snapshots (AD-5) |
| **Factory** | Deploys Rooms, gates capacity and Backstop | Creation-gate tests | Factory never custodies value beyond the creation transaction (AD-14) |
| **Room** | Owns all escrow and the waterfall | Full contract suite | No Room may reference another Room (AD-2) |
| **Indexer** | Consumes every contract event | Replay-equality suite | New event types must not break existing replay |
| **API** | Serves read models and the sole rate | Read-model and rate tests | Rate omission path must survive every change (AD-16) |
| **`shared/`** | Handle derivation consumed by app and indexer | Handle tests | A second implementation is a defect by definition (AD-17) |
| **`bindings/`** | Generated; consumed by app and services | Drift check | Regeneration is part of the build, never manual (AD-8) |

**Regression strategy:** the full contract suite runs on every PR and is the primary regression net. Any change to a contract's `#[contracterror]` enum requires confirming discriminants were not renumbered — codes are permanent once deployed, and clients map codes to copy.

---

## Appendix A: Code Examples & Tagging

**Contract tests** (the dominant surface) — ordinary Rust tests against `Env`:

```rust
#[test]
fn recipient_receives_full_pot_when_stake_covers_the_miss() {
    let ctx = RoomFixture::new()
        .members(10)
        .contribution(100_0000000)   // integer stroops, never floating point
        .started()
        .build();

    ctx.advance_to_round(3);
    ctx.member(4).skip_contribution();
    ctx.pass_grace_window();

    let recipient = ctx.payout_position_holder(3);
    let before = ctx.balance_of(&recipient);

    ctx.advance_round();

    assert_eq!(ctx.balance_of(&recipient) - before, ctx.full_pot());
    assert_eq!(ctx.stake_of(ctx.member(4)), ctx.expected_stake_after_one_slash(4));
    assert_eq!(ctx.backstop_drawn(), 0);   // Stake covered it; Backstop untouched
}
```

**Playwright tags for selective execution** (PWA layer):

```typescript
import { test } from '@seontechnologies/playwright-utils/api-request/fixtures';
import { expect } from '@playwright/test';

test('@P1 @API rate is omitted when outside the freshness window', async ({ apiRequest }) => {
  const { status, body } = await apiRequest({
    method: 'GET',
    path: '/api/rate',
    headers: { 'x-test-rate-age-seconds': '99999' },
  });

  expect(status).toBe(200);
  expect(body.rate).toBeUndefined();
  expect(body.source).toBeUndefined();
});
```

```bash
npx playwright test --grep "@P0|@P1"
```

```bash
cargo test --workspace
```

---

## Appendix B: Knowledge Base References

- **Risk Governance** (`risk-governance.md`) — P×I scoring, ≥6 mitigation, =9 gate FAIL
- **Test Priorities Matrix** (`test-priorities-matrix.md`) — P0–P3 criteria
- **Test Levels Framework** (`test-levels-framework.md`) — level selection and duplicate-coverage guard
- **Test Quality** (`test-quality.md`) — definition of done: no hard waits, <300 lines, <1.5 min
- **ADR Quality Readiness Checklist** (`adr-quality-readiness-checklist.md`) — the 29 criteria behind the testability review
- **NFR Criteria** (`nfr-criteria.md`) — NFR validation framing

---

**Generated by:** BMad TEA Agent
**Workflow:** `bmad-testarch-test-design`
**Version:** 4.0 (BMad v6)
