# Pal3 — Brief Addendum

Depth that belongs downstream (PRD, architecture, solution design) or earned a place but does not fit a 1–2 page brief.

> **Status 2026-08-08:** The brief references this file for (a) the full trust-model mechanism design and (b) the full regulatory risk register. Neither has been supplied yet — placeholders below mark what is owed. Do not treat the empty sections as "nothing to say"; treat them as known gaps.

---

## 1. Trust model — full mechanism design

**[NOT YET SUPPLIED]** The brief summarises four layers (real-ID KYC, network-wide trust score, slashable stake, underwriter backstop). The mechanism design owes answers to:

- How is the trust score computed? Inputs, weights, decay over time.
- What seeds a brand-new member's score from KYC alone?
- How does score map to payout position — strict ranking, banded, or weighted lottery?
- How is stake size determined? Flat, or inverse to trust score?
- Slashing: full stake on first miss, or graduated? Any cure period?
- Can a defaulter rehabilitate a score, and over what horizon?
- Sybil resistance beyond KYC — what stops one human, several identities?
- **Capacity-tier binding.** Room capacity unlocks by underwriter subscription tier post-MVP. What is the formula binding capacity to posted backstop capital, so a tier upgrade cannot outrun solvency? What happens to an in-flight room if an underwriter downgrades or lapses mid-cycle?
- **Payout position in large rooms.** Trust-ranked ordering is low-stakes in a 5-member room and high-stakes in a 50-member one, where last position means waiting most of a year. Does ordering need a fairness floor — a cap on consecutive late placements across cycles — once rooms scale?

## 2. Regulatory risk register (Philippines)

**[NOT YET SUPPLIED]** The brief names the headline exposures. The full register owes, per risk: the specific regulation, the trigger condition, the mitigation, and the counsel question.

Known headline items:
- SEC — investment-contract test applied to the underwriter's premium income.
- Insurance Commission — whether the backstop constitutes surety/insurance.
- BSP — VASP licensing freeze (Sept 2025) forcing reliance on a licensed VASP/EMI rail.
- AMLC — KYC/AML obligations once real identity and real money are in scope.

**The member-side argument (established 2026-08-08, use this in the application).** The SEC's stated objection to OnPals is the *promised return* — advisories cite 10%–757% over 1–90 days — which is what makes them Ponzi-type investment contracts. Pal3 promises members nothing beyond their own money returned on schedule. Member-side investment-contract exposure is therefore structurally near-zero, and the analysis narrows to the underwriter relationship alone. This is a design property, not a legal opinion, and it materially shrinks the surface counsel has to clear.

---

## 2b. Risks carried out of the brief

Trimmed from the brief's register at finalize to keep it to length. Still live, still owed answers.

- **Trust cold-start at the network level.** The score is only as strong as the identity behind it, so KYC quality is load-bearing. A weak identity layer makes the entire reputation graph decorative.
- **Category track record.** On-chain ROSCAs have a poor history. The narrative must explain why Pal3 survives where they didn't — trust engine, a credible regulated-rail path, and lean software-only revenue that keeps the platform out of the money flow.
- **Sybil resistance.** Covered under §1 as an open mechanism question, but it is also a standing risk: KYC is the only thing preventing one human from farming several identities to game payout order.

---

## 3. Chain & rail decision — options considered

Recorded 2026-08-08. Driven by the contradiction between the "regulated peso rails" differentiator and the stated Stellar deployment target.

**The facts:**

- **PHPC** (Coins.ph, first BSP-sandbox-approved PHP stablecoin) is issued on **Ronin** and **Polygon** — both EVM. There is no BSP-sanctioned peso stablecoin on Stellar as of Aug 2026.
- **Soroban** (Stellar's Rust/WASM contract platform, mainnet since Feb 2024) has **no native verifiable-randomness primitive**. The known path is a community relayer proxying Chainlink VRF from another chain — a cross-chain trust and liveness dependency under the "provably fair" claim.
- **Stellar Community Fund** Build awards run to ~$150K in XLM across Open / Integration / RFP tracks, on a rolling numbered-round cadence (SCF #43 closed 26 Apr 2026; later rounds follow).

**Options:**

| Option | Peso rail | Randomness | Grant fit | Cost |
|---|---|---|---|---|
| **A. Stellar/Soroban, as stated** | No PHP stablecoin — use USDC and carry FX friction, or wait for a PH anchor | Cross-chain relayer dependency | Strong — SCF eligible | Weakens the "regulated peso rails" differentiator, the one that separates Pal3 from the category graveyard |
| **B. EVM (Polygon), follow PHPC** | PHPC directly — differentiator intact | Chainlink VRF native | No SCF | Forfeits the grant that motivated the choice |
| **C. Build on Stellar, bridge/plan for PHP later** | USDC now, PH anchor when one exists | Same as A | Strong | Bets the core differentiator on a rail that may not arrive |
| **D. Drop VRF from V1** | Independent of chain | Deterministic tiebreak (e.g. trust score + join order), no randomness needed | Independent | Removes a dependency; costs the "provably fair draw" language. Note: trust-ranked payout already does the real work — VRF only breaks *ties* |

**Observation carried forward:** Option D is orthogonal to the chain choice and looks cheap. If payout order is set by trust score and ties are rare, a deterministic tiebreak may be sufficient for V1 and removes an entire class of infrastructure risk regardless of which chain wins.

**DECIDED 2026-08-08 (0xpayable): Option A, with V1 scoped as a controlled pilot.**

Build on Stellar/Soroban. Keep the SCF grant path. Redefine V1 from "real users" to a recruited pilot cohort operating under informed-consent terms, which contains the FX exposure rather than denying it. The regulated peso rail moves from a V1 differentiator claim to an explicit Phase 2 milestone.

Supporting facts gathered for this decision:

- **Category precedent at SCF exists.** *Lul* — remittances + ROSCA for refugees and the unbanked in Africa — was funded at ~$20,500 in XLM. SCF demonstrably funds savings-circle products.
- **Grant scale calibrated.** The ~$150K figure is the top of the Build range; category-comparable awards sit nearer $20–50K. Milestone funding for a pilot, not runway.
- **Coins.ph left Stellar.** They operated as a PHP Stellar anchor circa 2017 and issued PHPC on Polygon and Ronin. The most relevant company in the space chose EVM for peso rails. This must be a prepared answer, not a surprise, in any investor conversation.

**Consequences accepted:**

1. V1 traction evidence is weaker — a pilot proves the mechanism works, not that people want it.
2. Phase 2 is uncommitted: it depends on a peso stablecoin arriving on Stellar *or* a chain migration. The chain-agnostic trust engine is the hedge that keeps migration affordable.
3. VRF is dropped from V1 (Option D folded in) — deterministic tiebreak instead. Independent of chain; removes a cross-chain dependency at no meaningful cost.

---

## 4. Terminology change applied

"Investor" was doing two jobs in the source draft — the in-product role that underwrites a room, and the people who fund the company. Renamed the in-product role to **underwriter** throughout the brief. A pitch audience reading "investors earn per-contribution fees" would otherwise assume it meant *them*.
