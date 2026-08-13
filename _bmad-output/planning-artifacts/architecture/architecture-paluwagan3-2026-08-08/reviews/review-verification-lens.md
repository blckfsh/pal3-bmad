# Review — Verification Lens (update pass, 2026-08-12)

**Scope:** the AD-16 / AD-17 delta and the stack-table additions. Pre-existing ADs were verified in the 2026-08-08 run and are not re-litigated here.

**Verdict:** PASS with one note.

## Findings

**1. Client stack — verified against the web today, not asserted.** (no action)
Both claims underpinning the React + Vite + `vite-plugin-pwa` choice were checked on 2026-08-12:
- Stellar Wallets Kit is web-component based and documented for React, Angular, Vue, and vanilla JS — so the wallet library genuinely imposes no framework constraint, which is what made this an ecosystem-familiarity call rather than a technical one.
- Vite + `vite-plugin-pwa` is the current mainstream PWA path and the Create React App PWA template is deprecated — so the choice is not carrying a stale default.

**2. No versions pinned for React, Vite, or `vite-plugin-pwa`.** (note, consistent with existing posture)
The stack row says "pin versions at install." This matches the treatment already agreed for `@stellar/stellar-sdk`, `stellar-cli`, and Stellar Wallets Kit, and the memlog records the user's reason: nothing is being installed until the architecture is final, and pinning is a lockfile step at build time. The only architecturally load-bearing versions — `soroban-sdk` 25.0.0, Rust 1.84+/`wasm32v1-none` — remain verified. No action.

**3. FX rate provider is unnamed and correctly tagged.** (no action)
`[ASSUMPTION: a published USD/PHP source, not yet selected]` rather than a fabricated provider name. AD-16 makes the choice presentational, so nothing else in the spine binds to it. This is the right handling — naming a provider unverified would have been the failure.

**4. `signAuthEntry` finding from the original run remains accurate.** (no action, but see spine-external note)
AD-15 still correctly records that Stellar Wallets Kit's WalletConnect module exposes `stellar_signXDR` and `stellar_signAndSubmitXDR` but not `signAuthEntry`. The PRD's FR-21 still contains the superseded claim that `signAuthEntry` is required — a PRD-side fix, outside this spine.
