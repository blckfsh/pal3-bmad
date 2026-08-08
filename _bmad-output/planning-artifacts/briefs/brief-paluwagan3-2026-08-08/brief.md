---
title: "Product Brief: Paluwagan3 (Pal3) — on-chain stablecoin savings circle"
status: ready
created: 2026-08-08
updated: 2026-08-08
source: user-supplied draft dated 2026-06-20, reworked 2026-08-08
---

# Product Brief: Pal3

## Executive Summary

Pal3 is an on-chain, stablecoin-powered version of the Filipino rotating savings circle (ROSCA) — rebuilt so the failures that kill a paluwagan can't happen. In a traditional paluwagan a group pools money and each member takes the full pot in turn. It works until one person stops contributing or the organizer runs off with the cash, and then the whole circle collapses. One such collapse, Repa Paluwagan, cost investors roughly ₱2B across Davao and Bohol alone.

Pal3 replaces the human organizer with a smart contract that holds the funds, enforces every contribution, and pays out on a transparent, rule-based schedule — so organizer fraud is structurally impossible. It then answers member default with a trust engine: real-identity KYC, a network-wide reputation score that ranks who gets paid first, a small slashable stake, and an underwriter who insures the room against defaults for a fee.

V1 proves that trust engine as a controlled pilot on Stellar/Soroban. Migration to a regulated peso-stablecoin rail is the Phase 2 milestone.

## The Problem

Paluwagan is how millions of Filipinos already save and reach lump sums no bank would give them. It runs entirely on trust, and trust breaks in three places:

- **Member default.** One person takes the pot and stops contributing. Everyone after them loses. The circle dies and the social fallout is real.
- **Organizer fraud.** Someone must hold the money and run the rotation, and that person can vanish with the pool. This is the pattern behind the "Online Paluwagan" (OnPals) schemes the SEC keeps issuing advisories against.
- **No safety net, no record.** A defaulter faces no recovery and no reputational cost — they simply start a new circle elsewhere. Good savers earn no credit for being reliable.

A financial tool that is perfect for the capital-poor is throttled by the fact that you can only safely use it with people you already trust completely.

## The Solution

A mobile-first dapp where the smart contract *is* the organizer:

- **Members** join a room, complete KYC, lock a small stake, and contribute a fixed amount on a set cadence.
- **The contract** custodies every contribution, enforces the schedule, and releases the full pot to one member each round — order set by trust score, ties broken deterministically and auditably. No human ever touches the pool.
- **Default handling** is automatic: a missed contribution is covered first by slashing the defaulter's stake, then by the room's underwriter, so the scheduled recipient is always paid in full. The defaulter's reputation craters network-wide.
- **Stablecoins** hold value steady — the lesson taken from a prior volatile-token attempt.

## The Trust Model

The defining mechanism, and the reason Pal3 is not another on-chain ROSCA. Four identity-anchored layers:

1. **Real-ID KYC** seeds a member's starting trust score and makes default carry a real-world identity.
2. **A network-wide trust score** orders payouts and sizes stakes, and follows the member across every room.
3. **A slashable stake** covers the first tranche of any missed contribution.
4. **An underwriter backstop** makes the pot whole beyond the stake, for a per-contribution fee.

Two properties fall out of this design and are worth stating directly:

- **Organizer fraud becomes impossible**, not unlikely. The largest paluwagan scam vector is removed structurally rather than policed.
- **Trust-ranked payout inverts the core risk.** The most-proven members are paid earliest, so the people most able to default are the least likely to — concentrating underwriter exposure where uncertainty is lowest.

Full mechanism design in `addendum.md`.

## The Competitive Picture

The category splits on one question: **does the pot rotate, and who is liable when someone stops paying?**

| | Rotates | Funds enforced | Identity-anchored | Default covered | Regulated |
|---|---|---|---|---|---|
| **Traditional paluwagan** | Yes | No — human organizer | Social only | No | No |
| **GCash GSave / Maya Savings** | No — individual | Yes | Yes | N/A | Yes |
| **Tonik Group Stash** | No — each keeps their own | Yes | Yes | N/A | Yes |
| **On-chain ROSCAs** (HaloFi, WeTrust) | Yes | Yes | No — pseudonymous | No | No |
| **Pal3** | **Yes** | **Yes** | **Yes — KYC + trust score** | **Yes — stake + underwriter** | **Path stated** |

**The regulated incumbents don't rotate.** GSave, Maya Savings and Tonik's Group Stash are group *savings* — everyone contributes, everyone keeps their own. None delivers the lump sum that makes a paluwagan worth joining. They borrow the cultural label without the mechanism.

**The on-chain ROSCAs rotated but had no answer for default.** Under pseudonymous membership a defaulter's only cost is an abandoned wallet. That is why the category is a graveyard — a mechanism failure, not a market one, and precisely the gap the trust engine targets.

Pal3 is the only row with a Yes across all five. The October demo exists to substantiate that live.

## Market Opportunity

Three figures, deliberately kept apart.

**Serviceable today.** V1 requires employment records and bank details, so the addressable cohort is the formally employed: **32.96M wage and salary workers** (PSA, Jan 2026 — up 2.4M year on year), 77.6% of them in private establishments, average monthly wage ₱21,544. Documented, income-verifiable, enforceable by the trust engine.

**Reachable at scale.** **44% of Filipino adults remain unbanked**, chiefly on low income or missing documents — and account ownership *fell* to 50% in 2025 from 56% in 2021. The formal system is losing ground. This is the population the vision serves and V1 explicitly does not.

**The pain being priced.** Repa Paluwagan alone cost roughly **₱2B** across Davao and Bohol, against a continuing run of SEC advisories on OnPals schemes.

> **[ASSUMPTION — needs a defended number]** The reachable slice of that 32.96M is unset. Narrow it by what V1 actually demands: urban, smartphone-owning, and willing to hold a USD-pegged stablecoin for a full cycle. This is a sizing estimate for the deck, not an acquisition commitment — and it is a separate question from the pilot cohort below.

## Who This Serves

**Members — income earners.** Employed Filipinos (officemates, families, barangay groups, friend circles) who can supply employment records and bank details; verifiable income is what makes the trust engine enforceable. The launch persona is a salaried 25–40-year-old saving toward a specific goal — an allowance buffer, small capital, an emergency fund — who knows paluwagan and has either been burned or is afraid to join one.

**Underwriters.** Capital providers who open rooms, accept or reject members by trust score, and earn a per-contribution fee for absorbing default risk.

**Later.** Employer-sponsored rooms, then — as the trust graph matures — the informal-economy users in the vision.

## Revenue Model

- **Platform:** SaaS subscription for underwriters. Tiers unlock **more concurrent rooms and greater per-room member capacity** — V1 rooms are capped at pilot scale, with larger rooms opening at higher tiers post-MVP. This keeps Pal3 out of the pooled-money flow entirely: software, not a financial counterparty.
- **Underwriters:** per-contribution fees from members, the risk premium for insuring the room.
- **Members:** no yield, by design.

**Capacity tiers are a risk control, not only a paywall.** Room size drives underwriter exposure directly — more members means a larger pot, a longer cycle, and more default surface. Capping capacity by tier bounds how much risk any one underwriter can take on before they have demonstrated they can carry it, which is why the mechanism earns its place beyond monetisation.

One design constraint follows, and it matters for regulatory posture: **capacity must be gated on demonstrated capital adequacy, not on subscription payment alone.** If a higher tier lets an underwriter open a large room without a correspondingly larger posted backstop, the platform is selling permission to take risk that members ultimately bear. Tie the unlock to posted backstop capital and the tier becomes a feature licence sitting on top of a solvency check — defensible to a regulator, and honest to members.

That last line is a regulatory position, not an oversight. The SEC's objection to OnPals is the *promised return* — 10% to 757% — which is what makes them investment contracts. Pal3 promises members nothing beyond their own money back on schedule. Member-side exposure is therefore structurally near-zero, and the investment-contract question attaches only to the underwriter relationship.

## Core Technology

**V1 deploys on Stellar/Soroban** — Rust/WASM contracts for escrow, staking, slashing and payout. The trust engine itself (scoring, reputation, KYC linkage, default logic) is built **chain-agnostic**, as service code behind a thin on-chain interface, so the moat stays portable and the chain remains a swappable decision rather than a structural bet.

The choice is made with its trade-offs on the table:

- **Why Stellar.** A payments-first network whose mission — financial inclusion for the underserved — is Pal3's own. SCF is an active funding path with direct category precedent: *Lul*, a ROSCA product for the unbanked in Africa, was funded at ~$20.5K in XLM.
- **What it costs.** No BSP-sanctioned peso stablecoin exists on Stellar, so V1 runs USD-pegged and pilot members carry FX exposure for a full cycle. This is exactly why V1 is a consenting pilot and not a public launch.
- **No VRF dependency.** Soroban has no native verifiable randomness, and the only route is a cross-chain relayer to Chainlink VRF. Pal3 doesn't need it: trust-ranked ordering does the real work and randomness only ever broke *ties*. A deterministic, auditable tiebreak removes the dependency at no cost to any fairness claim that matters.

Full options analysis in `addendum.md`.

## Team

Solo founder. Four years of professional smart contract development (Solidity/EVM), migrating to Rust/Soroban for this build.

The load-bearing capability is contract engineering under adversarial conditions — escrow, custody of pooled funds, slashing logic, upgrade safety. That discipline transfers across execution environments; Soroban is a language migration on top of an established skill, not a first attempt at smart contracts.

Stated plainly: single-founder execution risk is real, and there is no in-house audit capability — which is why audit-readiness is a funded milestone rather than an assumption.

## Roadmap & Scope

Sequenced against the SCF #45 round cadence. Rounds run roughly every six weeks, with submission overlapping the review and voting phase.

| Phase | Window | Deliverable |
|---|---|---|
| Submission | → **16 Aug 2026** | SCF application + full MVP specification |
| Build | Aug–Sept 2026 | Working Soroban MVP on testnet |
| Demo & voting | ~Oct 2026 | Live demo to reviewers and community |
| Pilot | Post-award | Recruited cohort completes one full cycle |

**October demo — committed scope.** Deliberately narrow; three mechanisms that work beat six that half-work in front of the people voting.

- Room creation with configurable contribution amount and cadence
- Member join with stake lock
- Scheduled contributions held in contract escrow
- Trust-ranked payout of the full pot, deterministic tiebreak
- **One induced default → stake slashed → underwriter backstop tops up → scheduled recipient paid in full.** The centrepiece: the differentiator, executing live.

**Specified but not implemented for the demo:** production KYC/AML (mocked), underwriter subscription and onboarding (backstop present as a pre-funded pool), multi-room trust-graph effects, dispute resolution.

**Out of scope until Phase 2:** migration to a regulated peso rail, fiat on/off-ramps, open public enrolment. **Out until later:** multi-currency, employer-sponsored tier, subscription billing, dispute-resolution UI, legal recovery flow, secondary markets for trust or positions.

## The Pilot Cohort

Deliberately small, and not a user-acquisition exercise. One room, completing one cycle, proves the mechanism.

- **Size:** 5–10 members — the size of a real paluwagan room.
- **Cadence:** weekly. This is a scheduling decision, not a preference: a 5–10 member room at weekly cadence completes a full cycle in 5–10 weeks, while a monthly cadence would take 5–10 months and push the T3 milestone past any reasonable horizon.
- **Recruitment:** personal network — officemates, family, friends, an organised barangay group. Members join on informed terms, including the USD-peg exposure.
- **One underwriter**, which for the pilot may be the founder, so the two-sided cold start does not gate the first cycle.

**What this is really testing.** Not whether Pal3 can scale, but whether people will accept a small fee for a paluwagan that cannot collapse. If twelve people who already know and trust the founder will not join, that is decisive early evidence at near-zero cost — which is the point of running a pilot rather than a launch.

## Success Criteria

**V1 pilot:**

- One full cycle completed end-to-end by a recruited cohort, zero loss to any scheduled recipient.
- Participants join on informed terms — they understand funds are denominated in a USD-pegged stablecoin, not pesos, and accept that exposure for the pilot.
- At least one deliberately induced default, handled exactly as designed: stake slashed first, underwriter second, pot always whole.
- Default rate low enough that underwriter fees stay small — demonstrably cheaper and safer than a traditional paluwagan.
- Members return for a second cycle and trust scores visibly accrue.

**Beyond the pilot:** audited or audit-ready contracts, a counsel-reviewed regulatory path, and retention evidence — the three things that turn a working mechanism into a fundable company.

## The Ask

**SCF #45 Build Award — [ASSUMPTION: $45,000 in XLM]**, released against milestone tranches.

| Tranche | Released on | Allocation | [ASSUMPTION] |
|---|---|---|---|
| **T1** | Award | Founder engineering — 8-week MVP build to testnet | $18,000 |
| **T2** | Demo accepted | Security review / audit-readiness pass on escrow, stake, slash, payout | $15,000 |
| **T3** | Pilot cycle completed | PH fintech counsel opinion; KYC/AML integration; pilot cohort support | $12,000 |

Each tranche retires a named risk from this brief: T2 answers smart-contract security, T3 answers the regulatory question, T1 is the build. **Not requested:** marketing, salaries beyond the build window, or runway. Pal3 asks for the cost of proving the mechanism, not the cost of running a company.

> Calibration: the Build maximum is $150K, but category-comparable awards land nearer $20–50K. A solo founder requesting near the cap invites scrutiny the application won't survive. Figures above are placeholders pending real quotes.

## Risks & Open Questions

- **Regulatory (highest).** An underwriter earning a premium from a pool may implicate the SEC (investment contract) and/or the Insurance Commission (surety), and the BSP's freeze on new VASP licences (Sept 2025) means production likely runs on a licensed VASP/EMI rail. Needs PH fintech counsel — consciously framed, not solved. *Full register in `addendum.md`.*
- **The peso rail is a bet, not a booking.** Phase 2 depends on a BSP-sanctioned peso stablecoin reaching Stellar, or on Pal3 migrating to one. Neither is committed. *Expect the question: Coins.ph was a PHP Stellar anchor in 2017 and issued PHPC on Polygon and Ronin instead. That needs a prepared answer.*
- **Smart-contract security.** Escrow, staking, slashing and payout all hold real funds; an exploit would be terminal. Audit-readiness is a V1 requirement.
- **Fee pricing is an actuarial problem.** Too low and underwriters lose money; too high and members save nothing versus a free traditional paluwagan. Needs real default-rate data that does not yet exist.
- **Two-sided cold start.** A first cycle needs enough trustworthy members *and* a willing underwriter in the same room.
- **Pilot-stage traction is weak evidence.** A controlled cohort proves the mechanism works, not that people want it.
- **Single founder, eight-week window, new execution environment.** The schedule has no slack. Scope gets cut before the date moves.

## Vision

A Filipino who has never touched crypto joins a paluwagan with their officemates, saves safely for the first time without fear of being scammed, and builds a portable on-chain reputation that follows them everywhere — ending up holding and using stablecoins as naturally as GCash, without ever being lectured about blockchain. From there: employer-sponsored rooms, the informal economy, and a trust graph that becomes the financial reputation layer for the everyday Filipinos the formal system never served.
