# Story 1.2: Contract test harness and CI pipeline

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Member whose savings these contracts will hold,
I want every change to the contracts built, verified, and tested automatically,
so that the code holding my money is never changed without something checking it.

## Acceptance Criteria

**AC-1 — Contract unit tests with error-code assertion conventions**

**Given** the contract workspace
**When** the test harness is established
**Then** Soroban contract unit tests run against each contract using the SDK's test utilities, with conventions documented for asserting reverts by error code rather than by message (AR-20).

**AC-2 — Full-Cycle integration harness**

**Given** a Room under test
**When** an integration test exercises a full Cycle
**Then** the harness can form a Room, run every Round, induce a Default, resolve it through the waterfall, and close — with assertions available at each transition (SM-1, SM-2, SM-3).

**AC-3 — Ledger-time control, not wall clock**

**Given** a test that advances time
**When** it manipulates the clock
**Then** it advances the ledger timestamp rather than a wall clock, so Grace Window and Round deadline behavior is testable deterministically (AR-4, AR-18).

**AC-4 — CI builds, regenerates bindings, fails on drift, runs tests**

**Given** any change pushed to the repository
**When** CI runs
**Then** it builds the contract workspace, regenerates TypeScript bindings, fails if regenerated bindings differ from committed ones, and runs the full test suite (AR-8).

**AC-5 — Client and services type-checked against generated bindings**

**Given** CI
**When** it runs against the client and services
**Then** it type-checks and tests them against the generated bindings rather than hand-written contract types (AR-8).

**AC-6 — Deterministic suite**

**Given** a test suite run
**When** it completes
**Then** the result is deterministic — the same commit produces the same outcome, with no randomness or wall-clock dependence in any test (NFR-2, AR-11).

**AC-7 — Invariant properties are expressible in the harness**

**Given** the invariant properties that Story 5.3 asserts
**When** the harness is designed
**Then** it can express them as tests — no privileged transfer path, atomic waterfall revert, no randomness, expiry never as control flow — rather than leaving them to manual review (NFR-1, NFR-3, AR-6, AR-9).

## Tasks / Subtasks

- [ ] **Task 1 — Test module layout and shared fixture crate** (AC: 1, 2)
  - [ ] Add `contracts/shared/src/testutils.rs` behind a `testutils` feature, or a `contracts/test-support` dev-only crate — one home for fixtures, consumed by all three contract crates
  - [ ] Each contract crate gets `src/test.rs` (unit tests, `#[cfg(test)]`) for entry-point-level behaviour
  - [ ] Integration tests that span Registry + Factory + Room live in `contracts/room/tests/` (or the test-support crate), not inside a single contract's unit tests
  - [ ] Document the split in a short `contracts/TESTING.md`: unit = one contract, integration = multi-contract lifecycle

- [ ] **Task 2 — Error-code assertion convention** (AC: 1)
  - [ ] Establish `try_*` client methods as the required way to assert failure — never `#[should_panic]`, never string matching on messages (AR-20)
  - [ ] Provide an assertion helper, e.g. `assert_contract_err!(result, RoomError::NotAuthorized)`, so the pattern is one line at call sites
  - [ ] Document in `contracts/TESTING.md` with a worked example of a passing and a failing assertion
  - [ ] Add a lint or grep check rejecting `should_panic` in `contracts/**` — it cannot distinguish which error fired, and discriminant stability is the whole point of AR-20

- [ ] **Task 3 — Ledger-time control** (AC: 3, 6)
  - [ ] Fixture helpers wrapping `env.ledger()` to set and advance the ledger timestamp; advance sequence number alongside timestamp so TTL behaviour stays coherent
  - [ ] Provide named helpers over raw numbers: `advance_to_round_deadline()`, `advance_into_grace_window()`, `advance_past_grace_window()`
  - [ ] Prohibit `std::time`, `SystemTime`, `Instant`, and `chrono::now` anywhere under `contracts/` — add a grep check
  - [ ] Document that contracts never receive a client-supplied time (AR-18)

- [ ] **Task 4 — Authorization assertion convention** (AC: 1, 7)
  - [ ] **Critical:** provide a convention that pairs `env.mock_all_auths()` with an assertion on `env.auths()` — see Dev Notes, this is a correctness trap
  - [ ] Provide `assert_authorized_by!(env, address)` and a negative helper asserting a call fails when auth is absent (use `env.set_auths(&[])` or an unmocked env rather than mocking everything)
  - [ ] Provide a caller-class enumeration helper so a test can sweep {Member, non-member, Underwriter, Registry admin, stranger} against an entry point — Story 5.3 builds the full matrix on top of this
  - [ ] Document in `contracts/TESTING.md` why a bare `mock_all_auths` test is insufficient

- [ ] **Task 5 — Room lifecycle integration harness** (AC: 2)
  - [ ] Register a Stellar Asset Contract in tests via `env.register_stellar_asset_contract_v2` to stand in for the Room's stablecoin; mint balances to test Members
  - [ ] Builder API for a Room: member count, contribution amount, cadence, member Trust Scores, posted Backstop
  - [ ] Lifecycle drivers: `form_room()`, `fill_room()`, `start()`, `contribute_all(round)`, `skip_contribution(member)`, `advance_round()`, `close()`
  - [ ] Named fixture states reachable in one call: `filling`, `started`, `grace_open`, `default_resolved`, `member_removed`, `closed`
  - [ ] Balance-conservation helper asserting total value in equals total value out across any transition — reusable by every escrow test
  - [ ] The harness must be able to induce a Default and drive it through Slash → Backstop draw → Payout → Round advance, with assertions available at each step

- [ ] **Task 6 — Invariant expressibility for Story 5.3** (AC: 7)
  - [ ] Verify the harness can express, without further scaffolding: (a) no address can transfer escrow, alter a Payout Position after start, or reverse a Slash; (b) a forced failure partway through the waterfall reverts the whole invocation; (c) ordering, Stake sizing, and Slash amounts are pure functions of committed state and ledger timestamp; (d) no branch depends on entry expiry
  - [ ] For (b), provide a fault-injection seam — e.g. a token whose `transfer` can be made to fail on demand — so partial-failure paths are reachable
  - [ ] Write one thin smoke test per property proving the harness *can* express it. **Do not write the full invariant suite — Story 5.3 owns it.**
  - [ ] Record in `contracts/TESTING.md` which harness affordance each Story 5.3 acceptance criterion depends on

- [ ] **Task 7 — Determinism guarantees** (AC: 6)
  - [ ] No randomness in test inputs — no `rand`, no random ports, no UUIDs; fixed test addresses derived from fixed seeds
  - [ ] Deterministic iteration: prefer ordered collections over `HashMap`/`HashSet` wherever iteration order can reach an assertion
  - [ ] Run the suite twice in CI (or run with a shuffled test order) and confirm identical results
  - [ ] Document the rule: a flaky test is a defect in the test or the contract, never something to retry

- [ ] **Task 8 — CI pipeline** (AC: 4, 5, 6)
  - [ ] GitHub Actions workflow at `.github/workflows/ci.yml` — remote is `github.com/blckfsh/pal3-bmad`, so GitHub Actions resolves the `ci_platform: auto` setting
  - [ ] Install pinned Rust with the `wasm32v1-none` target, and pinned `stellar-cli`
  - [ ] Cache Rust artifacts with `stellar/actions/rust-cache@main`
  - [ ] Job steps in order: `stellar contract build` → regenerate bindings → **fail on any `bindings/` diff** → `cargo test --workspace` → `tsc --noEmit` across app/shared/services → TypeScript tests
  - [ ] Reuse Story 1.1's drift-check script rather than reimplementing the comparison in workflow YAML
  - [ ] Add the AD-15 static check: fail the build if any contract entry point requires a detached auth entry or multi-party authorization (see Dev Notes)
  - [ ] Add the hard-coded-config check and the `should_panic` / wall-clock checks from Tasks 2 and 3
  - [ ] Keep total PR runtime under ~15 minutes

- [ ] **Task 9 — TypeScript test setup** (AC: 5)
  - [ ] Add Vitest to `app/`, `shared/`, and each `services/*` package with a shared base config
  - [ ] One smoke test per package proving the runner works and that generated `bindings/` types import and type-check
  - [ ] Wire `test` and `typecheck` scripts at the workspace root so CI calls one command per concern
  - [ ] Do not add Playwright or any browser runner in this story

- [ ] **Task 10 — Documentation** (AC: 1, 3, 6)
  - [ ] `contracts/TESTING.md` covering: test layout, error-code assertions, auth assertions and the `mock_all_auths` trap, ledger-time helpers, fixture states, determinism rules, and the Story 5.3 affordance map
  - [ ] Keep it short enough to be read — this is a working reference, not a manual

## Dev Notes

### Hard dependency — read before starting

**Story 1.1 must be `done` before this story starts.** It is currently `ready-for-dev` and unimplemented; the repository still contains no code, no `Cargo.toml`, and no `package.json`. Everything here builds on 1.1's output:

| From Story 1.1 | Used here |
|---|---|
| Cargo workspace with `contracts/{registry,factory,room,shared}` | `cargo test --workspace` reaches every crate |
| `soroban-sdk` with `testutils` feature in `[dev-dependencies]` | The entire harness |
| One `#[contracterror]` enum per contract with explicit discriminants | Error-code assertion convention (Task 2) |
| Binding-drift check as a runnable script | CI calls it (Task 8) — do not reimplement |
| TypeScript workspace with `bindings/` wired as a dependency | Type-check against generated bindings (Task 9) |
| Stub entry points with a single `require_auth` on the transaction source | Auth assertions and the AD-15 check |

If any of the above is missing when this story starts, fix it in 1.1 rather than working around it here.

### The `mock_all_auths` trap — the single most important thing in this story

`env.mock_all_auths()` makes every `Address::require_auth` succeed as though authorization were provided. That is convenient and it is dangerous: **a test using `mock_all_auths` passes even when the contract is missing its `require_auth` check entirely.** The test cannot tell the difference between "authorization was correctly required and satisfied" and "no authorization was ever required."

For most projects that is a latent annoyance. Here it is a direct threat to the product's central claim. R-002 (privileged path to escrow, score 6) and R-007 (AD-15 conformance, score 6) are precisely the defects a naive harness would conceal, and AD-6 — "no address holds authority to transfer escrow" — is the invariant the entire organizer-fraud argument rests on.

**The convention this story must establish:** every test that calls `mock_all_auths` asserts the resulting authorization tree with `env.auths()` afterward, confirming the expected address was actually required to authorize. Negative tests must not use `mock_all_auths` at all — they should run against an env with no auth mocked (or `set_auths(&[])`) and assert the call fails.

Make this a documented, helper-backed convention rather than a note somebody has to remember. Story 5.3 builds its full negative-authorization matrix on top of it.

### AD-15 static check

AR-15/AD-15 requires that every entry point authorizes only the transaction source via a single `require_auth`, with no detached auth entry and no multi-party authorization in one invocation. This is not a style preference: Stellar Wallets Kit's WalletConnect module exposes `signXDR` and `signAndSubmitXDR` but not `signAuthEntry`, so an entry point needing detached auth falls outside the kit and makes the primary mobile surface unbuildable.

Implement the check at whatever fidelity is achievable now — at minimum a grep-level check for more than one `require_auth` in a single entry point, plus a test-level assertion that `env.auths()` contains exactly one entry for each entry-point invocation. Discovering a violation after the PWA is built means rearchitecting inside an eight-week window (R-007).

### Test level boundaries — what this story does not build

- **The invariant suite itself** — Story 5.3 owns it. This story proves the harness *can* express those properties and writes one smoke test per property, nothing more.
- **Contract behaviour** — Epics 2 and 3 own the waterfall, ordering, and escrow. The harness will initially drive stub entry points; that is expected, and the lifecycle drivers should be written so they light up as behaviour lands.
- **Playwright / E2E.** A framework-setup pass was deliberately halted pending Story 1.1's scaffold. E2E belongs after the PWA exists.
- **Backstop exposure simulation** — Story 5.5 owns it. The harness should make it *possible* (parameterized Room construction, balance conservation) without running the sweep here.

### Correction to the test design's task assignment

`test-design-architecture.md` assigns two testability blockers to Story 1.2. On closer sequencing, only one of them belongs here:

- **R-012, chain-state seeding harness** — the contract-side portion *is* this story (Task 5's named fixture states). The part that seeds a *deployed network* for API and PWA testing cannot be built until there is an API to test; it belongs with **Story 3.7** (event-sourced read models and query API). Build the fixture states here in a way a network-seeding script can later reuse.
- **R-011, wallet test-signer seam** — cannot be built here. There is no wallet integration until **Story 1.7** (wallet connection over WalletConnect), and no app shell until Story 1.3. The seam belongs with Story 1.7. It remains a blocker for E2E coverage; it is not a blocker for this story.

Neither risk is dismissed — both are reassigned to the story where the surface they attach to actually exists. Update the risk register owners accordingly if you re-run the test design.

### Soroban test API reference (verified 2026-08-13)

| Need | API |
|---|---|
| Test environment | `Env::default()` |
| Register a contract | `env.register(ContractType, ())` — `register_contract` is the older form |
| Ledger time | `env.ledger()` — set timestamp; use `with_mut` for full `LedgerInfo` including sequence number |
| Mock authorization | `env.mock_all_auths()` — **always pair with `env.auths()`**, see above |
| Assert required auths | `env.auths()` after the call |
| Assert error codes | `try_*` client methods, matching on the contract error enum |
| Token for escrow tests | `env.register_stellar_asset_contract_v2(admin)`, then `StellarAssetClient` to mint |
| Run inside contract context | `env.as_contract(&id, || { ... })` for direct storage inspection |

CI: `stellar/actions/rust-cache@main` caches Rust dependencies and build artifacts. `stellar contract build --print-commands-only` shell-escapes its output if you need to inspect or pipe the underlying commands.

### Determinism is an acceptance criterion, not an aspiration

AC-6 makes flakiness a build failure, and NFR-2 makes determinism a product property. Two consequences for how tests are written:

- Time only ever moves by explicit ledger manipulation. Any test that would pass or fail differently depending on when it runs is a defect.
- Iteration order matters. Ordering (FR-11) and score computation (FR-10) are both order-sensitive, and R-004 identifies map-iteration order as a specific non-determinism source to hunt. Prefer ordered collections anywhere iteration can reach an assertion.

### Coverage target

`test-design-qa.md` proposes ≥90% branch coverage on `contracts/room` and ≥80% elsewhere, flagged as **U-6, unratified**. Wire coverage reporting into CI so the number is visible, but do not enforce a failing threshold until the target is ratified. Reporting without gating is the useful halfway point.

### Project Structure Notes

- CI platform resolves to **GitHub Actions** — the remote is `github.com/blckfsh/pal3-bmad` and `ci_platform` is `auto` in `_bmad/tea/config.yaml`.
- `_bmad/` and `_bmad-output/` are committed planning artifacts. CI should not lint, test, or build them; exclude them from any workspace glob.
- No database, no Docker, no deploy configuration in this story — hosting is explicitly Deferred in the architecture spine.

### References

- [epics.md § Story 1.2](_bmad-output/planning-artifacts/epics.md) — acceptance criteria source
- [epics.md § Story 5.3](_bmad-output/planning-artifacts/epics.md) — the invariant properties AC-7 must support
- [epics.md § Additional Requirements](_bmad-output/planning-artifacts/epics.md) — AR-4, AR-6, AR-8, AR-9, AR-11, AR-15, AR-18, AR-20
- [ARCHITECTURE-SPINE.md § AD-4, AD-6, AD-8, AD-9, AD-11, AD-15](_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md)
- [prd.md § 10 Cross-Cutting NFRs](_bmad-output/planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md) — NFR-1, NFR-2, NFR-3
- [Story 1.1](_bmad-output/implementation-artifacts/1-1-project-scaffold-with-generated-contract-bindings.md) — the scaffold this builds on
- [test-design-architecture.md](_bmad-output/test-artifacts/test-design-architecture.md) — R-002, R-004, R-007, R-011, R-012
- [test-design-qa.md](_bmad-output/test-artifacts/test-design-qa.md) — P0 scenario set, execution strategy, coverage targets
- [soroban_sdk::Env](https://docs.rs/soroban-sdk/latest/soroban_sdk/struct.Env.html) — `mock_all_auths` / `auths` semantics
- [soroban_sdk::ledger::Ledger](https://docs.rs/soroban-sdk/latest/soroban_sdk/ledger/struct.Ledger.html) — timestamp control
- [Integrate Stellar Assets Contracts](https://developers.stellar.org/docs/build/guides/tokens/stellar-asset-contract) — SAC registration in tests
- [stellar/actions](https://github.com/stellar/actions) — `rust-cache` for CI

## Dev Agent Record

### Agent Model Used

### Debug Log References

### Completion Notes List

### File List
