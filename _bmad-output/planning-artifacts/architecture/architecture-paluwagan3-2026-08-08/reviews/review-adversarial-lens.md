# Review — Adversarial Lens (update pass, 2026-08-12)

**Method:** construct two units one level down that each obey every AD to the letter yet still build incompatibly.

**Verdict:** THREE HOLES FOUND in the new ADs. All three are the same shape — the AD constrains the *property* but not the *single source*, so two conforming implementations diverge.

## Critical

**A-1. Two conforming handle derivations. (AD-17)**
AD-17 requires the handle be "a pure deterministic function of their Stellar address, computed identically by every client and by the indexer," and defers the specific encoding to the UX spine. Construct the divergence: the PWA team implements a word-pair encoding from a SHA-256 of the address; the indexer team implements a base32 short code from the same address. **Both obey AD-17 perfectly** — each is pure, deterministic, address-derived, and internally identical everywhere it runs. Yet a Member's handle on the Members list would not match their handle in a round-detail record served by the API. In a product whose stated highest-harm failure is misreading who owes what, two names for one person is a money-legibility defect.

The Consistency Convention ("a second derivation is a defect") states the intent but does not prevent the build: two teams each believe they wrote *the* derivation.

*Fix: require one implementation, not two conforming ones — the derivation lives in a single shared module consumed by client and indexer alike.*

**A-2. The client can bypass the rate adapter and still obey AD-16. (AD-16)**
AD-16 says a rate adapter "fetches and caches a published rate and serves it through the API," and that the rate is never authoritative and never enters a contract decision. Construct the divergence: the PWA calls a public FX API directly at render time. It has violated nothing — the rate is still presentational, still carries a source and timestamp, still never touches a contract. But two Members opening the same terms screen seconds apart now see different peso figures for the same commitment, which is precisely the failure mode that caused the per-client-fetch option to be rejected in the first place.

*Fix: state the single path explicitly — clients obtain rates only from the API.*

## High

**A-3. Two places can decide staleness. (AD-16)**
AD-16 requires a peso line be "omitted rather than rendered stale and unlabelled," and the freshness window is deferred. Construct the divergence: the rate adapter treats a 6-hour-old rate as servable; the client treats anything over 1 hour as stale and suppresses it. Both obey the rule. The result is a UI that shows a peso line on one surface and not another, for the same rate at the same moment — and a debugging experience where nobody can say which layer dropped it.

*Fix: locate the decision in one layer. The API should omit the rate when it is outside the freshness window, so a client that receives no rate has nothing to decide.*

## Checked and clean

- **AD-16 against AD-1 and AD-7.** The rate adapter is read-only, never writes to a contract, and its unavailability degrades presentation only. It does not weaken AD-7's "off-chain services never adjudicate" — no money decision consults it.
- **AD-17 against AD-10.** A handle derived from an address introduces no personal data; it is a projection of a value already on-chain.
- **AD-16/AD-17 against inherited ADs.** Neither weakens nor contradicts AD-1 through AD-15.
- **No new entity acquires two owners.** The rate is owned solely by the rate adapter; the handle is owned by nothing — it is computed, not stored.
