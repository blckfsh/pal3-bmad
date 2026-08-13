---
title: 'TEA Test Design → BMAD Handoff Document'
version: '1.0'
workflowType: 'testarch-test-design-handoff'
inputDocuments:
  - '_bmad-output/test-artifacts/test-design-architecture.md'
  - '_bmad-output/test-artifacts/test-design-qa.md'
  - '_bmad-output/planning-artifacts/epics.md'
  - '_bmad-output/implementation-artifacts/sprint-status.yaml'
sourceWorkflow: 'testarch-test-design'
generatedBy: 'TEA Master Test Architect'
generatedAt: '2026-08-13'
projectName: 'paluwagan3'
---

# TEA → BMAD Integration Handoff

## Purpose

Bridges the test design outputs into BMAD's implementation planning so risk assessments and test requirements flow into stories rather than sitting beside them.

**Sequencing note.** The template assumes this document is consumed by `create-epics-and-stories`. On this project that workflow has already run — `epics.md` holds 5 epics and 45 stories, implementation readiness is `READY`, and `sprint-status.yaml` is generated. This handoff is therefore **retrofit guidance for `bmad-create-story`**: each story file created from `epics.md` should absorb the acceptance criteria below as it is prepared. No epic restructuring is proposed or required.

## TEA Artifacts Inventory

| Artifact | Path | BMAD Integration Point |
|---|---|---|
| Architecture test design | `_bmad-output/test-artifacts/test-design-architecture.md` | Blockers, risk register, NFR testability requirements |
| QA test design | `_bmad-output/test-artifacts/test-design-qa.md` | Story acceptance criteria, coverage plan, execution strategy |
| Risk assessment | Embedded in the architecture document | Story priority and epic-level quality gates |
| Coverage strategy | Embedded in the QA document | Story test requirements |
| Working progress record | `_bmad-output/test-artifacts/test-design-progress.md` | Workflow state and full reasoning trail |

## Epic-Level Integration Guidance

### Risk References

| Epic | Risks carried | Epic-level quality gate |
|---|---|---|
| **Epic 1** — verified member with a working wallet | R-005, R-007, R-010, R-011, R-012 | Story 1.2 must deliver the test-signer seam, the seeding harness, and the AD-15 static check, or every later epic inherits an untestable surface |
| **Epic 2** — a room you can read before you commit | R-002, R-009, R-014, R-020 | Negative-authorization matrix exists and passes for every Factory and Room entry point added in this epic; Stake rounding specified before Story 2.5 |
| **Epic 3** — a cycle that pays out and covers a miss | R-001, R-002, R-003, R-006, R-013, R-021 | Full-Pot assertion green in all five default branches; waterfall atomicity fault injection green |
| **Epic 4** — trust that compounds | R-004, R-008, R-018 | Independent reproduction (Story 4.5) matches on every recorded history, sharing no code with `services/trust` |
| **Epic 5** — pilot readiness | R-001, R-006, R-015, R-016, R-019 | **R-001 closed, not waived**, before the mainnet gate; audit-readiness review passed |

### Quality Gates

- P0 pass rate 100%, no waivers, at every epic boundary
- P1 pass rate ≥95% with failures triaged and explicitly accepted
- Every risk at score ≥6 mitigated or formally waived (owner, reason, expiry) before the mainnet pilot
- R-001 (score 9) closed by simulation, not waived — this alone blocks the mainnet gate
- Contract branch coverage ≥90% on `contracts/room`, ≥80% elsewhere `[PROPOSED — U-6, needs ratification]`
- Binding regeneration produces zero diff

## Story-Level Integration Guidance

### P0 Scenarios That Must Become Story Acceptance Criteria

These are not a separate test backlog. Written as acceptance criteria they cannot be descoped independently of the feature they protect — which matters because test work and feature work compete for one person here.

| Story | P0 scenarios to embed as acceptance criteria |
|---|---|
| 1.2 Contract test harness and CI pipeline | P0-017 (AD-15 static check); harness, signer seam, seeding harness as deliverables |
| 1.4 Registry attestations with duplicate rejection | P0-011, P0-012 |
| 1.6 KYC provider webhook issues attestations | P0-011 (adapter field allowlist) — blocked until the interface is frozen |
| 2.1 Factory deploys a Room with validated parameters | P0-003 (Factory entry points) |
| 2.2 Backstop posting gates Member admission | P0-003, P0-006 |
| 2.5 Member join with score snapshot and Stake lock | P0-010, P0-013, P0-015 |
| 2.6 Room start with Fairness Floor ordering | P0-009, P0-016 |
| 2.7 Open-window cancellation | P0-020 |
| 3.1 Per-Round Contribution escrow with Underwriter fee | P0-006, P0-021 |
| 3.2 Permissionless Round advance with full-Pot payout | P0-001, P0-014 |
| 3.3 Grace Window opens and accepts a cure | P0-019 |
| 3.4 Atomic Slash, Backstop draw, and Payout on Default | P0-001, P0-002, P0-005, P0-007 |
| 3.5 Stake exhaustion removes a Member | P0-001, P0-007 |
| 3.6 Room close returns Stakes | P0-022 |
| 4.1 Deterministic, versioned Trust Score computation | P0-009 |
| 4.5 Independent score reproduction | P0-008 |
| 5.3 Invariant verification | P0-003, P0-004, P0-005, P0-018 |
| 5.5 Backstop adequacy verified | P0-007 — **the gate-blocking scenario** |

### Testability Requirements for Story Preparation

- **Error codes before copy.** Every contract's `#[contracterror]` enum must be defined with stable discriminants before any client story consumes it. Clients map codes to copy and never parse messages; renumbering a deployed discriminant is prohibited.
- **`data-testid` on state-bearing UI.** The states worth targeting are those a test must distinguish: grace banner and its remaining-time element, per-Member contribution status chips, payout position and date, Stake and fee lines on the terms screen, the FX acknowledgement control, and the funding-shortfall message. Selector resilience matters most on the grace banner — the highest-consequence screen in the product.
- **Named fixture states.** Story preparation should reference the seeding harness's named states (filling, started, grace-open, default-resolved, member-removed, closed) rather than describing setup prose per story.
- **Balance conservation as a habit.** Any story touching escrow should assert total value in equals total value out across the transition, not only the intended leg.

## Risk-to-Story Mapping

| Risk ID | Category | P×I | Recommended Story / Epic | Test Level |
|---|---|---|---|---|
| R-001 | BUS | 3×3=**9** | 5.5 (verification), 3.4 / 3.5 (behaviour) | Contract (parameterized sweep) |
| R-002 | SEC | 2×3=6 | 5.3, plus every contract story in Epics 2–3 | Contract |
| R-003 | TECH | 2×3=6 | 3.4 | Contract (fault injection) |
| R-004 | DATA | 2×3=6 | 4.1, 4.5 | Service + Contract |
| R-005 | SEC | 2×3=6 | 1.4, 1.6 | Contract + Service |
| R-006 | TECH | 2×3=6 | 1.1 (storage conventions), 5.3 | Contract + on-chain soak |
| R-007 | TECH | 2×3=6 | 1.2 (CI check), 1.7 (real path) | Contract + CI |
| R-008 | BUS | 2×3=6 | 3.4 / 3.12 (instrumentation), retrospective | Service |
| R-009 | DATA | 3×2=6 | 2.5 | Contract |
| R-010 | OPS | 3×2=6 | 1.6, 1.8 | Blocks Service |
| R-011 | OPS | 3×2=6 | 1.2 | Blocks E2E |
| R-012 | OPS | 3×2=6 | 1.2 | Blocks API + E2E |
| R-013 | BUS | 2×2=4 | 3.3, 3.11 | Contract + Component |
| R-014 | TECH | 2×2=4 | 2.5 | Contract |
| R-015 | OPS | 2×2=4 | 5.3 | On-chain drill |
| R-016 | PERF | 2×2=4 | 5.4 | On-chain |
| R-017 | DATA | 2×2=4 | 3.7 | Service |
| R-018 | TECH | 3×1=3 | 2.6 | Contract (simulated) |
| R-019 | SEC | 1×3=3 | Accepted for pilot; 5.7 review | None |
| R-020 | TECH | 3×1=3 | 2.8 | Service + Component |
| R-021 | SEC | 1×3=3 | 3.4, 5.7 | Contract + review |

## Blockers Feeding Back Into Sprint Planning

Five items must be resolved before the stories that depend on them can be prepared. Three of them land on **Story 1.2**, which is second in the sprint plan — so they are due almost immediately.

| Blocker | Gates | Owner |
|---|---|---|
| Wallet test-signer seam (R-011) | Story 1.2 → all E2E | Founder/Dev |
| Chain-state seeding harness (R-012) | Story 1.2 → all API/PWA state coverage | Founder/Dev |
| KYC adapter interface frozen (R-010) | Stories 1.6, 1.8 | Founder |
| Stake rounding specified (R-009 / U-5) | Story 2.5 | Founder (product) |
| Transaction-cost threshold ratified (U-1) | Story 5.4 | Founder |
| Corrected Backstop formula (R-001) | Story 5.5, mainnet gate | Founder (product) |

Contract-level P0 work depends on none of these and can start as soon as the harness exists.

## Recommended BMAD → TEA Workflow Sequence

1. **TEA Test Design** (`TD`) — complete, produced this handoff
2. **TEA Framework** (`TF`) → **TEA CI** (`CI`) — recommended next, and worth folding into Story 1.2 rather than running standalone, since that story already owns the harness and pipeline
3. **BMAD Create Story** (`CS`) — absorbs the acceptance criteria above as each story is prepared
4. **TEA ATDD** (`AT`) — red-phase acceptance tests per story, most valuable on Epic 3's waterfall stories
5. **BMAD Dev Story** (`DS`) → **Code Review** (`CR`)
6. **TEA Automate** (`TA`) → **TEA Trace** (`TR`) — coverage completeness before the mainnet gate
7. **TEA NFR Assess** (`NR`) — once implementation evidence exists; it will need the UNKNOWN thresholds ratified first

## Phase Transition Quality Gates

| From Phase | To Phase | Gate Criteria |
|---|---|---|
| Test Design | Story Creation | All risks at score ≥6 have a named mitigation and owner; the five pre-implementation blockers are assigned |
| Story Creation | ATDD | Stories carry the P0 acceptance criteria mapped above |
| ATDD | Implementation | Failing acceptance tests exist for all P0/P1 scenarios in the story's scope |
| Implementation | Test Automation | All acceptance tests pass; binding drift is zero |
| Test Automation | Testnet | P0 100%, P1 ≥95%; negative-authorization matrix complete |
| Testnet | Mainnet pilot | **R-001 closed by simulation**; Stories 5.3–5.5 passing; audit-readiness review passed (T2); counsel confirmation in hand (T3) |
