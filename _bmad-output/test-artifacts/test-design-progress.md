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
  - '_bmad/tea/config.yaml'
  - 'resources/knowledge/adr-quality-readiness-checklist.md'
  - 'resources/knowledge/nfr-criteria.md'
  - 'resources/knowledge/test-levels-framework.md'
  - 'resources/knowledge/risk-governance.md'
  - 'resources/knowledge/test-quality.md'
  - 'resources/knowledge/test-priorities-matrix.md'
---

# Test Design Progress — paluwagan3

## Step 1: Mode Detection & Prerequisites

**Workflow mode:** Create (no prior progress file or outputs in `_bmad-output/test-artifacts/`)

**Test design mode:** **System-Level**

**Reasoning:**

- Rule A (user intent, highest priority): both PRD/architecture AND epics/stories exist → prefer System-Level first.
- File-based detection (Rule B) would have indicated Epic-Level because `sprint-status.yaml` exists, but Rule A takes precedence.
- Practical driver: Epic 1 Story 1.2 ("Contract test harness and CI pipeline") will be implemented directly from these outputs. Framework selection, test levels, and NFR thresholds must be settled above the epic line so that story has a strategy to build against.
- Epic-Level passes remain available afterward, per epic, once the system-level plan exists.

**Prerequisites (System-Level) — all satisfied:**

| Requirement | Status | Source |
|---|---|---|
| PRD (FRs + NFRs) | Present, status `final` | `planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md` |
| ADRs / architecture decisions | Present, 17 ADs (AD-1…AD-17) | `planning-artifacts/architecture/.../ARCHITECTURE-SPINE.md` |
| Architecture / tech-spec | Present | Same spine + `SCF-architecture-outline.md` (grant-facing narrative, not build substrate) |
| Epics (for scope) | Present, 5 epics / 45 stories | `planning-artifacts/epics.md` |

**No HALT conditions triggered.**

**Notable project condition:** This is a greenfield repository — planning artifacts only, no source code, no package manifests, no existing tests. Consequences carried forward:

- Stack detection cannot rely on manifest scanning; the stack is read from the architecture spine's decisions rather than inferred from the filesystem.
- No existing test coverage to analyze and no running application to explore via browser automation.
- The test design is fully prescriptive (defining what will be built) rather than partly descriptive (assessing what exists).

---

## Step 2: Context & Knowledge Loading

### Configuration resolved

| Key | Value | Note |
|---|---|---|
| `test_artifacts` | `{project-root}/_bmad-output/test-artifacts` | |
| `test_design_output` | `_bmad-output/test-artifacts/test-design` | Handoff document location |
| `tea_use_playwright_utils` | `true` | Applies to the PWA layer only — see stack note below |
| `tea_use_pactjs_utils` | `false` | Contract testing between services is not the shape of this system |
| `tea_pact_mcp` | `none` | |
| `tea_browser_automation` | `auto` | No running app to explore — browser exploration skipped |
| `tea_execution_mode` | `auto` | |
| `test_stack_type` | `auto` | |
| `risk_threshold` | `p1` | |

### Stack detection

Filesystem scan found **no manifests of any kind** — no `Cargo.toml`, `package.json`, `playwright.config.*`, `pyproject.toml`. Auto-detection is therefore inapplicable and `{detected_stack}` is resolved from the architecture spine instead:

**`detected_stack` = `fullstack` (planned, not yet materialized)**, comprising three distinct test surfaces:

1. **Rust / Soroban contracts** (`contracts/registry`, `contracts/factory`, `contracts/room`, `contracts/shared`) — soroban-sdk 25.0.0, Rust 1.84+, target `wasm32v1-none`. This is where all authoritative logic and all value live.
2. **TypeScript services** (`services/trust`, `services/kyc`, `services/indexer`, `services/rates`, `services/api`) plus `shared/` and generated `bindings/`.
3. **React + Vite PWA** (`app/`), wallet access via Stellar Wallets Kit → WalletConnect.

**Consequence for tooling, recorded explicitly:** the TEA templates and `playwright-utils` assume an API/UI-centric system. Here the dominant test surface is the Rust contract suite running under `soroban_sdk::Env`, which is where the money-safety invariants are proven. Playwright applies to surface 3 and to cross-surface journeys only. The coverage plan below reflects that weighting rather than the template's default.

### Browser exploration

Skipped. `tea_browser_automation` is `auto`, but no application exists to explore and no URL is available. No CLI sessions were opened, so none require cleanup.

### Existing test coverage analysis

None to analyze. No `tests/`, `spec`, `e2e`, or `api` directories exist. No fixture or test patterns are established — every convention in this plan is being set for the first time, which is an advantage: there is no legacy to work around.

### Knowledge fragments loaded

System-Level required set, all five loaded:

- `adr-quality-readiness-checklist.md` — 8 categories / 29 criteria, used for the testability review
- `nfr-criteria.md` — NFR validation framing
- `test-levels-framework.md` — level selection
- `risk-governance.md` — P×I scoring, ≥6 mitigation, =9 gate FAIL, TECH/SEC/PERF/DATA/BUS/OPS categories
- `test-quality.md` — definition of done for tests

Also loaded: `test-priorities-matrix.md` (P0–P3 criteria) — normally Epic-Level required, loaded here because the coverage plan assigns priorities.

Not loaded, with reasons: `probability-impact.md` (Epic-Level required; scoring definitions already carried by `risk-governance.md`), Pact/contract-testing fragments (`tea_use_pactjs_utils` false; the system has no consumer/provider HTTP contract topology — the client talks to contracts through generated bindings, which is a compile-time contract, not a runtime one), `playwright-cli.md` (no app to drive), webhook fragments (deferred to the epic-level pass on Epic 1, where the KYC webhook is actually built).

### Persistent facts

`customize.toml` declares `persistent_facts = ["file:{project-root}/**/project-context.md"]`. No `project-context.md` exists anywhere in the project — the glob resolves empty. No facts loaded, no override applied. (`bmad-generate-project-context` would populate this once code exists.)

### Extraction summary

**Tech stack and dependencies:** as above. Version-pinning status is uneven — `soroban-sdk` is pinned at 25.0.0 and Rust at 1.84+, but `stellar-cli`, `@stellar/stellar-sdk`, Stellar Wallets Kit, React/Vite are all marked `[ADOPTED] — pin at install`. PostgreSQL and the FX provider are `[ASSUMPTION]`.

**Integration points:** KYC provider → `services/kyc` (webhook, inbound); FX rate provider → `services/rates` (poll, inbound); Stellar network → contracts + indexer; WalletConnect → PWA; SAC token contract → Room escrow transfers.

**NFRs identified:** funds safety (dominant), determinism, no privileged override, availability at Round boundaries, cost per Round, DPA/privacy compliance. Thresholds extracted and gaps recorded in Step 3 §3.

---

## Step 3: Testability Review, Risk Assessment & NFR Planning

### 3.1 Testability Concerns (actionable first)

Assessed against the 29-criterion ADR quality readiness checklist across controllability, observability, and reliability.

**TC-1 — No wallet test-signer seam (controllability). Severity: blocking for UI E2E.**
Every money-moving action in the PWA is a signed transaction reached through Stellar Wallets Kit's WalletConnect module. WalletConnect's handshake is a human-in-the-loop pairing flow against an external wallet app; it is not automatable as-is. Nothing in the spine defines an alternative signing path for tests. Without a seam — a test-only signer that satisfies the same interface and signs with a known keypair — no automated end-to-end test can traverse join, contribute, or any other value-moving journey. Manual testing is the only fallback, which does not scale to a regression suite. Feeds R-011.

**TC-2 — No chain-state seeding harness (controllability). Severity: blocking for UI/API testing of mid-Cycle states.**
Checklist criterion 1.3 (State Control). The interesting states are deep: "Round 5 of 10, Member 3 inside an open Grace Window, Member 7 removed by Stake exhaustion, Backstop drawn twice." Reaching them requires driving real contract invocations in sequence with ledger-time manipulation. In Rust contract tests this is free (`Env` gives direct ledger control). Against a deployed network for API and PWA testing, it requires a scripted seeding harness that nothing currently owns. Feeds R-012.

**TC-3 — KYC provider unnamed (controllability + contract ambiguity). Severity: blocks two stories.**
Checklist criteria 1.1 (Isolation) and 1.4 (Sample Requests). The PRD and spine both refer to "the KYC Provider" as an abstraction; no provider is named anywhere, including the Assumptions Index. Consequently the webhook payload shape, the duplicate-detection signal, the tier vocabulary, and the retry/idempotency semantics are all undefined. Story 1.6 (webhook issues attestations) and Story 1.8 (verification handoff) cannot be built to a real contract, and a stub cannot be built faithfully either — a stub invented now will encode guesses that the real provider contradicts. Feeds R-010.

**TC-4 — Soroban state archival is only partly reproducible in test (controllability). Severity: moderate.**
AD-3 mandates persistent storage with TTL extension on every write; AD-4 forbids expiry as control flow. Both are testable in `Env` via ledger sequence manipulation, but the archival/restoration behaviour of a live network under a genuinely stalled Room is not faithfully reproduced by the test harness. The spine's own Deferred list concedes a stalled Room has no cheap rescue path. This is the invariant most likely to pass in test and fail on-network. Feeds R-006.

**TC-5 — No observability requirements for off-chain services (observability). Severity: low for pilot, real for operations.**
Checklist category 6 (Monitorability). The spine specifies nothing about logs, metrics, tracing, or correlation IDs for `services/indexer`, `services/api`, `services/trust`, or `services/rates`. Nor does the spine decide hosting (explicitly Deferred). For a one-Room pilot this is survivable; it means that when the indexer drifts, diagnosis is archaeology. Feeds R-017.

**TC-6 — Availability and latency thresholds absent (observability of "good"). Severity: moderate.**
Checklist criteria 3.3 (SLA Definitions) and 7.1 (Latency). The PRD asserts "Contribution and Payout must succeed at scheduled boundaries" and that cost must be "negligible" — neither is a number, so neither is a pass/fail criterion. Recorded as UNKNOWN thresholds in §3.3 rather than guessed.

**TC-7 — Fairness Floor is structurally unexercisable in V1 (reliability of coverage). Severity: latent.**
FR-11's floor triggers on "the final Payout Position in the two immediately preceding Cycles within the same Room." V1 runs one Room for one Cycle. The rule therefore cannot fire in the pilot and will ship with zero production evidence. It must be covered by simulated multi-Cycle unit tests or it is untested code shipping into cohort 2. Feeds R-018.

**TC-8 — Stake formula is real-valued over an integer domain (controllability of expected values). Severity: moderate.**
`Stake = Contribution × (2 − TrustScore/1000)` in a domain where the consistency conventions mandate integer stroops and forbid floating point. Rounding direction, operation order, and precision are unspecified. A test cannot assert an expected value that the specification does not determine, and two faithful implementations can disagree. Feeds R-009.

### 3.2 Testability Assessment Summary (what is already strong)

This architecture is markedly more testable than typical for its domain, and several ADs are effectively testability decisions:

- **AD-11, no randomness anywhere.** Every outcome is a pure function of committed state and ledger timestamp. This is the single largest testability asset in the system: ordering, Slash amounts, and score changes are all exactly assertable, and property-based and golden-file testing become viable rather than aspirational.
- **AD-4, ledger timestamp as the sole time source, compared against deadlines in state.** Time is an input, not an ambient. `Env` sets it directly. Every time-dependent branch — Round boundary, Grace Window, open window — is deterministically reachable without sleeps or clock mocking. This eliminates the flakiness class that usually dominates scheduled-system test suites.
- **Errors convention: one `#[contracterror]` enum per contract, stable discriminants, never renumbered.** Tests assert on codes, not strings. Negative-path assertions stay stable across refactors.
- **Events convention: every value-moving transition emits an event, and the indexer is built only from events, never state polling.** Read models are replayable by construction, which makes "replay the same event sequence twice, assert identical models" a cheap and high-value test. It also gives the contract suite a clean observability surface.
- **AD-2 per-Room contract instances.** Natural test isolation with no shared-state bleed; parallel safety comes free at the contract level.
- **AD-8 generated bindings, hand-written encoding prohibited.** Client/contract type drift becomes a build-time failure instead of a runtime money bug. A CI check that regeneration produces no diff is cheap and closes the class.
- **AD-17 handle derivation in exactly one shared module.** One implementation means one test target; the "two conforming derivations produce two names" defect is designed out rather than tested for.
- **AD-13 permissionless `advance_round`, idempotent, caller-independent.** Trivially testable from any address, and it removes the keeper from the liveness path.

### 3.3 Architecturally Significant Requirements (ASRs)

| ASR | Source | Statement | Classification |
|---|---|---|---|
| ASR-1 | AD-9 | Slash, Backstop draw, and Payout resolve in one invocation — fully succeed or fully revert | **ACTIONABLE** |
| ASR-2 | AD-11 / NFR | No randomness; every outcome reproducible from chain history by any observer | **ACTIONABLE** |
| ASR-3 | AD-6 / NFR | No address holds authority to move escrow, alter a Position after start, or reverse a Slash | **ACTIONABLE** |
| ASR-4 | AD-13 | `advance_round` callable by any address, idempotent, reverts if early | **ACTIONABLE** |
| ASR-5 | AD-15 | Every entry point authorizes the transaction source via a single `require_auth`; no detached auth entry; no multi-party auth in one invocation | **ACTIONABLE** (statically checkable) |
| ASR-6 | AD-3 / AD-4 | Value-bearing state is persistent, TTL extended on every write; expiry is never control flow | **ACTIONABLE** |
| ASR-7 | AD-10 / §9 | No personal data on-chain — addresses, opaque provider reference, tier, integers only | **ACTIONABLE** |
| ASR-8 | AD-16 | API is the sole rate source; omits the rate outside the freshness window; rate never reaches a contract | **ACTIONABLE** |
| ASR-9 | AD-5 | Score snapshotted at join; join reverts if Registry no longer matches the displayed score | **ACTIONABLE** |
| ASR-10 | AD-8 | Bindings generated from built contracts; regeneration is part of the build | **ACTIONABLE** (CI check) |
| ASR-11 | AD-7 | Total off-chain outage leaves every in-flight Room fully operable by direct invocation | **ACTIONABLE** (drill) |
| ASR-12 | NFR §10 | Cost per Round negligible against a ₱1,000-equivalent Contribution | **ACTIONABLE** (threshold UNKNOWN) |
| ASR-13 | AD-1 | Ledger is the only system of record; off-chain stores are rebuildable caches | FYI (structural; enforced by AD-7 tests) |
| ASR-14 | AD-17 | Handle derived from address by exactly one shared module | FYI |
| ASR-15 | AD-12 | Contract source published Apache 2.0 from first deployment | FYI (Story 5.1 task, not a test) |

### 3.4 Risk Register

Full register with mitigations, owners, and timelines is in `test-design-architecture.md`. Summary of scoring:

- **21 risks identified** — 1 critical (score 9), 11 high (score 6), 9 medium (score 3–4), 0 low.
- **Critical:** R-001, Backstop adequacy formula known to be insufficient against its own worst case.
- **Category distribution:** TECH 7, SEC 5, DATA 4, OPS 4, BUS 3, PERF 1. (R-001 spans BUS/DATA; counted under BUS.)
- Three of the high risks (R-010, R-011, R-012) are **testability blockers** rather than product defects — they block the building of tests, not the running of the system. They are scored at impact 2 for that reason, and they are the items with the earliest deadlines because they gate Story 1.2.

### 3.5 NFR Planning — thresholds and gaps

**In scope:** Security, Reliability, Performance/Cost, Compliance, Maintainability, Availability.

**Thresholds successfully extracted (defined, testable):**

| NFR | Threshold | Source |
|---|---|---|
| Grace Window duration | 48 hours | FR-14 |
| Room open window | 14 days before cancellation | FR-6 `[ASSUMPTION]` |
| Trust Score domain | 0–1000, clamped after every update | FR-10 |
| Score deltas | +10 / +2 / −150 / +50 / −400 | FR-10 `[ASSUMPTION]` |
| Starting scores | 300 basic, 400 full; cap 400 until one Cycle completes | FR-2 `[ASSUMPTION]` |
| Stake formula | Contribution × (2 − TrustScore/1000) | FR-13 `[ASSUMPTION]` — rounding UNKNOWN |
| Minimum Backstop | 2 × Contribution × Member count | FR-19 `[ASSUMPTION]` — **disputed, see R-001** |
| Capacity | 1 Room, max 10 Members | FR-18 `[ASSUMPTION]` |
| Slash amount | Exactly the missed Contribution, not the whole Stake | FR-15 |
| Recipient outcome | Full Pot, every branch, no delay, no reduction | FR-12/FR-15 — binary |
| Determinism | Reproducible from event history by any observer | NFR §10 — binary |
| Privileged override | None, under any code path | NFR §10 — binary |
| Personal data on-chain | None | AD-10 / §9 — binary |
| Audit readiness | Review passed before any mainnet Room with real value | NFR §10 / T2 — gate |

**UNKNOWN thresholds — recorded, not guessed:**

| # | Missing threshold | Why it matters | Disposition |
|---|---|---|---|
| U-1 | Maximum acceptable transaction cost per Round | "Negligible relative to ₱1,000" is not a pass/fail criterion. The PRD's own worked example (a $0.50 fee on ~$18 being "close to 3%") reads as a rejection, not a limit. Story 5.4 measures cost but has nothing to measure it against. | Clarification item → R-016. Needs a number (percentage of Contribution, or absolute) before Story 5.4 can pass or fail. |
| U-2 | Availability target for Contribution/Payout at Round boundaries | NFR asserts they "must succeed" with no target, window, or retry expectation. | Clarification item. Partly mitigated by AD-13 (any caller can advance) — arguably the architectural answer is "liveness is permissionless, so no service SLA is required," which would make this N/A rather than unknown. Confirm. |
| U-3 | Indexer/API uptime and read-model staleness bound | Members fund decisions from read models. No freshness bound is stated. | Clarification item → R-017. |
| U-4 | FX rate freshness window | AD-16 fixes that the API alone decides staleness and omits the rate when stale, but not the threshold value. Spine defers this to the UX spine; DESIGN.md/EXPERIENCE.md were not re-read for a value in this pass. | Clarification item → R-020. Owed before Story 2.8. |
| U-5 | Stake rounding semantics | Real-valued formula, integer domain. | → R-009. Must be specified, not chosen at implementation time. |
| U-6 | Contract test coverage target | Nothing specifies a coverage bar. | Proposed in the quality gates below; needs ratification. |
| U-7 | API P95/P99 latency | Unspecified. | Deferred with hosting (spine Deferred list). Not a pilot blocker at cohort size <12. |
| U-8 | Read-model rebuild time (replay RTO) | AD-1 makes stores rebuildable; how long a rebuild takes is unmeasured. | Clarification item, low priority at pilot scale. |

**Planned evidence sources:** contract test suite reports (`cargo test`), independent score-reproduction harness output (Story 4.5), exposure-table simulation output (Story 5.5), invariant verification report (Story 5.3), measured cost report (Story 5.4), audit-readiness review (Story 5.7), CI binding-drift check, negative-authorization suite results, event-replay determinism results.

**Boundary respected:** this step plans NFR validation only. No PASS/CONCERNS/FAIL status is assigned — that belongs to `nfr-assess` once implementation evidence exists.

---

## Step 4: Coverage Plan & Execution Strategy

### 4.1 Test levels selected

| Level | Tooling | What it owns | Why here |
|---|---|---|---|
| **Contract tests** (dominant) | Rust, `soroban_sdk::Env`, `cargo test` | Every invariant that moves money or changes obligation: waterfall, ordering, stake sizing, authorization negatives, storage/TTL, time transitions | This is where authoritative state lives (AD-1). Deterministic, fast, no network, ledger time directly controllable. |
| **Service integration** | TypeScript, Vitest | Indexer event replay, trust score computation and reproduction, API read models, rate adapter freshness, KYC webhook adapter | Component boundaries with real behaviour worth testing; fast enough for PR. |
| **Binding/contract-drift** | CI check | `stellar contract bindings` regenerates with no diff; generated clients compile against services and app | Closes AD-8's defect class at build time (checklist 8.2 analogue). |
| **Component (UI)** | Vitest + Testing Library | Screen states — grace banner, terms disclosure, funding shortfall, members list, rate-omitted rendering | Cheaper and more stable than E2E for state permutations. |
| **E2E** | Playwright (+ `playwright-utils`) | A small set of critical journeys only: verify → connect → join → contribute → receive payout | Requires TC-1 and TC-2 resolved. Kept deliberately thin — E2E is for journeys, not logic. |
| **On-chain acceptance** | Scripted testnet runs | Epic 5: invariant verification, cost measurement, exposure adequacy | Some claims are only true on a real network. |

Duplicate-coverage guard applied: waterfall arithmetic is proven **only** at the contract level and asserted nowhere else; UI tests assert rendering of state, never derive it; E2E asserts journey completion, never amounts.

### 4.2 Coverage summary

| Priority | Count | Share |
|---|---|---|
| P0 | ~22 | ~18% |
| P1 | ~35 | ~28% |
| P2 | ~32 | ~26% |
| P3 | ~10 | ~8% |
| Supporting (fixtures, harness, seeding) | ~25 | ~20% |
| **Total** | **~124 scenarios** | |

**Justified deviation from the <10% P0 guidance.** The `test-quality` best practice caps P0 at under 10% of scenarios. This plan runs ~18%. The reason is that the P0 set here is almost entirely *invariant* tests rather than *feature* tests: they assert the properties on which the product's central claims rest (no privileged path to funds, recipient always paid in full, every outcome reproducible). In a system that custodies members' money with no recovery mechanism and no privileged override, an invariant that is merely P1 is an invariant that can regress silently into a mainnet deployment. The deviation is deliberate and recorded rather than accidental.

### 4.3 Execution strategy

**Philosophy:** run everything in PRs unless the infrastructure genuinely forbids it. Rust contract tests execute in seconds and Playwright parallelizes hundreds of tests into 10–15 minutes, so there is no case for tiering functional tests by priority.

- **Every PR (~10–15 min target):** full `cargo test` contract suite; all TypeScript service tests; binding-drift check; UI component tests; the thin E2E journey set against a local Stellar quickstart network.
- **Nightly (~30–60 min):** testnet deployment smoke; measured transaction-cost run (U-1 evidence); full Backstop exposure-table parameter sweep — every default pattern across a 10-Member Room, which is combinatorially too large for PR latency; extended property-based runs on ordering and score.
- **Weekly (~hours):** high-iteration invariant fuzzing; off-chain outage drill (ASR-11) — bring down indexer, API, trust and rate services, then complete a full Round by direct contract invocation; TTL/archival soak against a deliberately stalled Room (TC-4).

**Manual, excluded from automation:** audit-readiness review (Story 5.7); counsel confirmation of regulatory posture; licence/publication verification (Story 5.1); pilot cohort recruitment.

### 4.4 Effort estimate (ranges)

| Priority | Effort | Notes |
|---|---|---|
| P0 | ~60–90 h | Invariant and negative-authorization suites; waterfall branch matrix; exposure simulation harness |
| P1 | ~50–80 h | Grace/removal/fee/capacity mechanics, replay determinism, API contract, key screens |
| P2 | ~25–45 h | Edge cases, UI state permutations, secondary surfaces |
| P3 | ~8–16 h | Benchmarks, exploratory, documentation validation |
| Supporting infrastructure | ~30–50 h | Test-signer seam, chain-state seeding harness, factories, CI wiring |
| **Total** | **~175–280 h** | **~4.5–7 weeks, one engineer full-time** |

**Framing note.** There is no dedicated QA engineer on this project. The estimate is *test development effort*, and most of it lands inside story implementation rather than a separate track — Story 1.2 owns the harness and CI, and each subsequent story carries its own tests. Against an eight-week build window this is substantial but not disproportionate for a funds-custody system; it is the reason Story 1.2 sits second in Epic 1.

### 4.5 Quality gates

| Gate | Threshold |
|---|---|
| P0 pass rate | 100% — no exceptions, no waivers |
| P1 pass rate | ≥95%, with every failure triaged and explicitly accepted |
| High risks (score ≥6) | All mitigated or formally waived with owner, reason, and expiry before mainnet |
| Critical risk (score 9) | R-001 must be closed — not waived — before any mainnet Room opens with real value |
| Contract code coverage | ≥90% branch on `contracts/room`; ≥80% elsewhere `[PROPOSED — U-6, needs ratification]` |
| Service/app coverage | ≥80% `[PROPOSED — U-6]` |
| NFR evidence | Every in-scope NFR category has a named evidence artifact produced; final PASS/CONCERNS/FAIL deferred to `nfr-assess` |
| Binding drift | Zero diff on regeneration |
| Testnet gate | Stories 5.3, 5.4, 5.5 complete and passing before the mainnet pilot |

Full P0–P3 scenario tables are in `test-design-qa.md`.

---

## Step 5: Output Generation & Validation

### Execution mode

Resolved to **sequential**. `tea_execution_mode` is `auto` and `tea_capability_probe` is `true`; parallel generation of the two documents was available but not used, because the architecture and QA documents share a risk register and a threshold list that must stay numerically identical across both. Reconciling two independently generated documents costs more than writing them in sequence.

### Outputs written

| Document | Path | Lines |
|---|---|---|
| Architecture test design | `_bmad-output/test-artifacts/test-design-architecture.md` | 424 |
| QA test design | `_bmad-output/test-artifacts/test-design-qa.md` | 539 |
| BMAD handoff | `_bmad-output/test-artifacts/test-design/paluwagan3-handoff.md` | 149 |

### Checklist validation

Validated against `checklist.md`. Passing:

- Prerequisites: PRD, ADRs (17), architecture, epics all present; requirements testable
- Risk assessment: 21 risks, unique IDs, categories assigned, P and I each 1–3, scores correct, ≥6 flagged, mitigations specific with owners and timelines, residual risk documented in Accepted Trade-offs
- NFR planning: categories identified; thresholds extracted; 8 marked UNKNOWN with none guessed; each converted to a risk or clarification item; evidence sources named; boundary respected — no PASS/CONCERNS/FAIL assigned
- Coverage: atomic scenarios, levels selected, duplicate-coverage guard applied and stated, P0–P3 assigned, risk links present
- Execution strategy: simple PR / Nightly / Weekly, philosophy stated, no re-listing of tests, no priority-tier structure
- Estimates: all ranges, no false precision, setup included
- Quality gates: P0 100%, P1 ≥95%, ≥6 mitigation required, coverage target proposed and flagged as needing ratification
- Two-document structure: actionable-first ordering, Quick Guide three tiers, testability concerns above the assessment summary, no test code or scenario tables in the architecture document, no quality gates or tool-selection sections there either
- Cross-document consistency: all 21 risk IDs present and identical in all three documents; dates, author, PRD and ADR references match
- Handoff: artifact inventory, epic-level risk references, P0-to-story acceptance-criteria mapping, full risk-to-story table, phase gates
- CLI sessions: none opened, none to clean up
- Artifacts: all under `{test_artifacts}/`

### Known deviations, recorded rather than silently passed

1. **Architecture document length.** The checklist targets ~150–200 lines; the document is 424. The same checklist mandates a full mitigation plan — strategy, owner, timeline, status, verification — for every risk at score ≥6, and there are twelve of those. The two requirements are in direct tension at this risk density. Length was allowed to grow rather than dropping mandatory mitigation plans or under-scoring risks to reduce their count. The Quick Guide exists precisely so the document is navigable at this size.
2. **P0 share of total scenarios.** ~18% against a <10% best practice. Justified in both output documents: the P0 set is composed of invariants, not features, in a system that custodies member funds with no privileged override and no recovery path.
3. **Template stack assumption.** The TEA templates and `playwright-utils` presume an API/UI-centric system. The dominant surface here is a Rust contract suite under `soroban_sdk::Env`. Playwright examples are retained for the PWA layer, where they apply; the coverage plan is weighted to where the money actually is.

### On-complete hook

`workflow.on_complete` resolves to an empty string in `customize.toml` with no team or user override present. No hook executed.
