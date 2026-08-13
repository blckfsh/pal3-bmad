---
name: Pal3
description: Information architecture, behavior, states, and flows for the Pal3 V1 pilot — member app and role-gated underwriter surfaces.
status: final
created: 2026-08-11
updated: 2026-08-12
sources:
  - "{planning_artifacts}/prds/prd-paluwagan3-2026-08-08/prd.md"
  - "{planning_artifacts}/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md"
---

# Pal3 — Experience Spine

> Behavior, IA, states, interactions, accessibility, and flows. Visual identity lives in `DESIGN.md`, referenced here by `{token}` name. Where a mock, wireframe, or import disagrees with either spine, the spine wins.

## Foundation

Single surface: a mobile-first PWA (PRD §12), rendering in the browser on Android Chrome and iOS Safari. No native app, no app-store review. Wallet connection is WalletConnect (FR-21), which means the signing experience involves leaving the PWA for a wallet app and returning — that round trip is a first-class design concern, not an implementation detail.

Member and underwriter are the same application, role-gated: after wallet connection the app resolves whether this address underwrites a room and reveals the underwriter surfaces if so. V1 has exactly one underwriter (the founder), so this is the cheapest correct answer, not a scaling decision.

English only. Light mode only. `DESIGN.md` is the visual identity reference.

**Implementation substrate, adopted 2026-08-12:** React + Vite + `vite-plugin-pwa`. This is a rendering framework and a build tool — it supplies no components, so nothing in this spine inherits from it and the component tables below stay absolute rather than collapsing to deltas.

**No component system is adopted, decided 2026-08-12.** shadcn, MUI, Radix, and React Aria were all considered and declined: the visual language is deliberately sparse — no shadows, one accent, outline-only pills — so a styled library's defaults would be overridden on nearly every component, and headless primitives were judged not worth the dependency for eleven components.

That decision moves real work onto the build, and this spine records it rather than leaving it implicit: **focus traversal, ARIA wiring, dialog semantics, and the `aria-live` grace banner are hand-built.** The Accessibility Floor below is therefore a build risk carried by a solo developer, not a property inherited from tested primitives. It is the section most likely to be quietly under-delivered under time pressure, and the one to check first if the eight-week window tightens.

## Information Architecture

| Surface | Reached from | Purpose | FRs |
|---|---|---|---|
| **Welcome** | Cold open, unauthenticated | What Pal3 is, in four lines. One action: connect a wallet. | — |
| **Connect wallet** | Welcome | WalletConnect session establishment. | FR-21 |
| **Verify identity** | After connect, no attestation | Explains what verification is for, hands off to the KYC Provider. | FR-1, FR-2 |
| **Verification status** | Return from provider | Pending / verified / failed / duplicate. Shows resulting starting score. | FR-1, FR-2, FR-3 |
| **Room terms** | Invitation link; Home when unjoined | The commitment screen. Complete terms on one screen. | FR-22, FR-13, FR-20 |
| **Funding check** | Room terms → commit | Balance vs. stake + first contribution; exact shortfall if any. | FR-21 |
| **Commit** | Funding check | FX acknowledgement, then a single signed transaction locking stake + contribution. | FR-22, FR-13, FR-8 |
| **Room filling** | After commit, before start | Members joined / needed, open-window countdown, cancellation terms. | FR-6 |
| **Room home** | Tab bar (default) | Current round, my obligation, my payout date, schedule. The app's centre of gravity. | FR-23, FR-9, FR-12 |
| **Contribute** | Room home | Pay the current round's contribution. | FR-8 |
| **Members** | Room home tab | Every member's status this round, by handle. | FR-9 |
| **Round detail** | Schedule row tap | One round's outcome: who was paid, what it cost, what resolved. | FR-12, FR-15 |
| **Trust score** | Tab bar | Score, what it buys, and every event that moved it. | FR-23, FR-10, FR-13 |
| **Cycle summary** | Room close | Final record: rounds completed, stake returned, score change. | FR-7 |
| **Account** | Room home header | Handle, verification tier, wallet session, disconnect. | FR-1 |
| **Activity** | Account | Every on-chain action this address took, with explorer links. | — |
| **Create room** | Underwriter home | Contribution, cadence, member count, within tier. | FR-4, FR-18 |
| **Post backstop** | Create room | Required backstop computed and locked before admission opens. | FR-19 |
| **Applicants** | Underwriter room | Review and admit or reject, on Pal3 history only. | FR-5 |
| **Underwriter room** | Tab bar (underwriter role) | Live and maximum exposure, contributions received, open grace windows. | FR-17, FR-14 |

→ Composition reference: [`mockups/room-terms-commit.html`](mockups/room-terms-commit.html) (Room terms unacknowledged; Commit funded and acknowledged). Spine wins on conflict.

Bottom tab bar, three tabs for a member: **Room** / **Trust** / **Account**. Underwriters get a fourth: **Underwrite**. No drawer. Sheets stack one level deep, never two.

**Closure note.** Every surface above except **Account** and **Activity** is landed by a PRD journey (UJ-1…UJ-4). Those two exist because a WalletConnect session and an irreversible on-chain history need somewhere to live, not because a stated need produced them — flagged rather than back-filled with an invented journey. Likewise, FR-16 removal-after-stake-exhaustion has a specified state below but no journey of its own; it is currently the far edge of UJ-3.

## Voice and Tone

Microcopy only. Brand posture lives in `DESIGN.md § Brand & Style`.

The governing rule: **plain language, honest mechanics**. Never volunteer "stablecoin", "smart contract", "escrow", or "blockchain" — but never imply money is somewhere it isn't, and never soften an irreversible consequence. The vocabulary is available on tap (Activity, explorer links, the terms screen's mechanics section), never in the path of someone trying to pay ₱1,000.

| Do | Don't |
|---|---|
| "Your money is held by the contract until your payout date." | "Funds are secured in escrow." |
| "You'll get $144.00 on 6 October. Nothing can change that date." | "Guaranteed payout!" |
| "You have 47 hours to pay before your stake is used." | "Payment overdue" |
| "Dennis paid late. Everyone still got paid on time." | "Issue resolved ✓" |
| "This is held in US dollars. The peso value can go up or down before your payout." | Burying FX in a checkbox label |
| "You missed round 4. $18.00 was taken from your stake. Your score dropped 150." | "Your account has been updated." |
| Name the person: "Marivic covers any shortfall." | "The underwriter provides coverage." |
| Sentence case, full sentences, no exclamation marks. | Encouragement, congratulation, or reassurance the mechanics don't earn. |

Numbers are never rounded in commitment copy. "About ₱1,000" is acceptable for an indicative peso line; "$18.00" is exact everywhere.

## Money Legibility

*Product-specific. The single largest source of potential harm in this UI is a member misreading what they owe or what they'll receive.*

- **The dollar figure is the number of record**, everywhere, always. It is what the contract holds and what the member signs.
- **An indicative peso line may appear beneath it** in `{typography.meta}`, always prefixed `≈`, never inside a button label, never on the FX acknowledgement itself. Decided 2026-08-11: comprehension for a cohort that thinks in pesos outweighs the risk of implying a stability the product disclaims — on the condition that the attribution rule below is honored without exception.
- **The peso rate source and its timestamp are stated** on every commitment surface carrying a peso figure. Attribution is stated **once per surface, beneath the primary amount, and governs every peso figure on that surface** — repeating it on each terms row would be noise, but a surface with no attribution anywhere shows no peso figures at all. An unattributed conversion is a promise. Both parts are required: a timestamp without a named source is not attribution.
- **A peso line is never stale.** The freshness window is **one hour** (decided 2026-08-12). Per `ARCHITECTURE-SPINE.md` AD-16 the API omits an out-of-window rate entirely, so the client receives nothing and renders no peso line — the decision lives in one layer and the UI never has to judge it. One hour was chosen over tighter windows because a comprehension aid that intermittently vanishes on a poor connection is worse than one slightly behind but reliably present; members must never be trained to expect it missing.
- **When the peso line is absent, nothing takes its place.** No placeholder, no "rate unavailable" copy, no dash. The dollar figure is the number of record and stands alone perfectly well — an error string beside an amount would imply something is wrong with the amount.
- **Four amounts must never be confusable** and are always labeled with their own noun: *contribution* (what you pay each round), *stake* (locked collateral, returned at close), *pot* (what you receive on your round), *fee* (what the underwriter earns per contribution).
- **The pot is always shown as its full value with no deduction**, because FR-12 and FR-20 guarantee exactly that. Any UI that shows a "net" payout would misrepresent the product.
- **Stake is always shown as a multiple and an amount together** — "1.6× one contribution — $28.80" — because the multiple is what the trust score controls and the amount is what leaves the wallet.

## Irreversibility & Commitment

*Product-specific. Pal3 has no admin override, no dispute process, and no way to reverse a slash (PRD §5, §10). The UI is the last point at which anything can be prevented.*

Three actions are irreversible and share one pattern: **join** (locks stake + first contribution), **contribute** (escrows funds for the round), and **admit an applicant** (fixes a member into a room that becomes immutable at start).

The commit pattern, in order, on one screen:

1. What you are about to give up, as an amount.
2. What you get and when, as an amount and a date.
3. What happens if you can't pay later — the grace window, the slash, the score penalty — stated before commitment, not after.
4. `{spacing.commit-gap}`.
5. One primary action, labeled with the act and the amount: "Lock $46.80 and join". Never "Confirm", never "Continue".

→ Composition reference: [`mockups/room-terms-commit.html`](mockups/room-terms-commit.html), state B. Spine wins on conflict.

There is no undo, so there is no undo affordance and no toast offering one. There is also **no confirmation dialog** — a second modal asking "are you sure?" trains dismissal and adds nothing over a screen whose entire purpose is the decision. The disclosure carries the weight.

**Removal (FR-16) is disclosed at join**, not discovered at removal. A member must have read, before locking anything, that an exhausted stake removes them from the room and forfeits an unreceived payout.

## Trust Score Legibility

*Product-specific, and directly measured: SM-5 targets ≥80% of the cohort correctly stating their score and why it changed.*

The score surface is built to be recited, not admired.

- **Score, consequence, history — in that order, always together.** The number alone is meaningless; the panel states "Your stake is 1.6× one contribution" immediately beneath it.
- **Every event is a plain sentence with its delta**: "Paid round 3 on time · +10 · 24 September". No icons, no categories, no aggregation.
- **The next threshold is stated in member terms**, not score terms: "40 more points and your stake drops to 1.5×" — because members act on the stake, not the integer.
- **The score is never a badge, level, tier, streak, or grade**, and never appears as a decoration next to a handle. It appears on the Trust surface, on the terms screen where it sets the stake, and nowhere else.
- **A default entry is permanent and says so** at the moment it is written: "This stays on your record." The PRD requires members be told before joining that this history cannot be erased (§9); the same sentence appears on the terms screen and in the event row.
- **Other members' scores are not shown to members.** Underwriters see them (FR-5); members see only contribution status by handle (FR-9). Nothing in the member UI ranks the cohort against each other.

## Component Patterns

Behavioral. Visual specs live in `DESIGN.md § Components`.

| Component | Use | Behavioral rules |
|---|---|---|
| Terms row | Room terms, funding check, cycle summary | Read-only. Never truncates a value; wraps instead. Tapping a row with a mechanic behind it opens a plain-language sheet. |
| Amount display | Commit surfaces, payout, pot | Dollar primary, indicative peso secondary. Never animates or counts up. |
| Status pill | Members list, schedule, underwriter room | Text carries the state; color reinforces. Four states only: paid, pending, in grace, defaulted. |
| Schedule row | Room home, cycle summary | Tap → round detail. Current round marked; member's own round set strong. Never reorders — position is frozen at start (FR-6, FR-11). |
| Member status row | Members | Handle only. No avatar, no name, no photo — the product does not hold legal identity and must not imply it does. The handle is a short alphanumeric code derived from the wallet address (AD-17), identical everywhere it appears, never chosen or changed by the member, and never accompanied by copy implying it is a name. |
| Trust score panel | Trust, room terms | Score + fill + consequence, never fewer than all three. |
| Notice banner | Room home | Two variants: grace (own or another member's) and default-resolved. Persistent, not dismissible, cleared by state change only. |
| Disclosure block | Room terms, commit | Full ink, never collapsed behind "read more", never scrolled past silently — the commit control stays disabled until the FX acknowledgement is checked (FR-22). |
| Button (primary) | One per surface | Labeled with act + amount on any funding action. Disabled state states the blocker in text beneath it, never as a tooltip. |
| Button (secondary) | Beside a primary action, or alone on a navigational surface | Never commits funds and never carries an amount in its label. Used for "not now" and for leaving a commitment surface without acting. Never styled or placed to compete with the primary action, and never the only control on a surface that has something to commit. |
| Signing handoff | Any transaction | See § Interaction Primitives. |

## State Patterns

| State | Surface | Treatment |
|---|---|---|
| Unauthenticated | Welcome | Four lines and one action. No marketing scroll. |
| **Trust score, no history yet** | Trust score | The state every member is in at pilot start, and the surface SM-5 measures. Score and its consequence render normally; the history area states plainly that nothing has moved the score yet and names the first thing that will — the next on-time contribution. Never an illustration, never an encouragement. |
| Activity, nothing yet | Activity | A member who has taken no on-chain action. One line stating that actions will appear here once they act. No placeholder rows. |
| Applicants, none yet | Applicants | An underwriter whose room has opened admission but drawn no requests. States that admission is open and how many members the room still needs — the useful fact, not "no results". |
| Wallet connected, unverified | Verify identity | What verification is for, who holds the documents, what Pal3 never sees (FR-1). |
| Verification pending | Verification status | Provider is working. No countdown Pal3 can't honor. App remains usable; joining is blocked with the reason stated. |
| Verification failed / duplicate | Verification status | State the outcome and the provider's remedy path. Pal3 offers no appeal, and says so rather than implying one exists (FR-3). |
| Insufficient balance | Funding check | Exact shortfall, in dollars, with the required total broken into stake + first contribution (FR-21). Primary action disabled; blocker stated in text. |
| Room filling | Room filling | X of N joined, days remaining in the open window, and the cancellation guarantee: everything returned in full if it doesn't fill (FR-6). |
| Room cancelled | Room filling → terminal | What was returned, to which address, with the transaction link. No apology framing — the guarantee worked. |
| Round open, unpaid (mine) | Room home | Amount, deadline, primary action. The only screen state where the primary action is a payment. |
| Round open, paid (mine) | Room home | Confirmed with date. Next obligation and its date stated immediately — the app never leaves a member wondering when they're next due. |
| **Grace window (mine)** | Room home | Grace banner in `{colors.state-grace-wash}`, persistent, with hours remaining and the exact consequence at expiry: the amount that will be slashed and the 150-point penalty. Primary action becomes the payment. This is the highest-stakes state in the product (see § Notification & Reach). |
| Grace window (another member's) | Room home, Members | Room state reads "awaiting contribution" with time remaining (FR-14). Never names blame; the handle is shown, the judgement isn't. |
| Default resolved | Room home, Round detail | Plain statement: who missed, what covered it, that the recipient was paid in full and on time (FR-15). Present as the system working, because it is. |
| Removed (stake exhausted) | Room home → terminal | The member's own removal, stated completely: rounds remaining now covered by the backstop, payout forfeited if not yet received, score penalty applied (FR-16). No path back in V1, and the copy does not imply one. |
| Payout received | Room home, Round detail | Full pot, date, transaction link. No celebration, no animation — it arrived because it was always going to (UJ-4). |
| Cycle complete | Cycle summary | Stake returned, score change, and the concrete carry-forward: what this score means for the next room's stake. |
| Transaction submitted | Any signing surface | Pending state with the action named. Never blocks the whole app; the member can navigate away and return. |
| Transaction failed | Any signing surface | Plain cause where knowable (rejected in wallet / insufficient balance / network), and an unambiguous statement of whether money moved. This sentence matters more than any other error string in the product. |
| Wallet session expired | Any | Reconnect prompt in place. Read-only surfaces remain readable — a member should never lose sight of their schedule because a session dropped. |
| Offline | Any | Last-known state with an "as of" timestamp. Contribution and signing disabled with the reason stated. Never show stale money state without its timestamp. |

## Notification & Reach

*Product-specific, and a named, deferred risk rather than a solved problem.*

V1 ships **in-app notification only** (founder decision, 2026-08-11). FR-14's two required notifications — grace-window open and 24-hours-remaining — are therefore delivered as room-home state and a persistent grace banner, reaching only a member who opens the app.

This is the product's weakest link, and the spine records it as such: the 48-hour grace window ends in an automated slash and a 150-point penalty, and in V1 the only warning lives on a screen the member has no external prompt to open. Mitigations inside the chosen constraint:

- The grace banner is the loudest element the design system permits, is not dismissible, and survives navigation across every surface — not only Room home.
- The room terms screen states the deadline mechanic before commitment, so the window is never a surprise even if the notification is never seen.
- Round deadlines are stated as absolute dates and times, never as relative countdowns alone, so a member can put them in their own calendar. `[ASSUMPTION: an "add to calendar" affordance on the schedule is the cheapest external reach available without infrastructure — worth building if the eight-week window allows.]`

**Deferred escalation, to switch on before the mainnet pilot opens with real value:** SMS at grace-open and at 24-hours-remaining, as the only channel that reaches a member whose salary is late and whose app is closed. Recorded here so it is a decision to revisit, not a discovery to make after someone is slashed.

## Interaction Primitives

- **Tap to act.** No long-press actions, no swipe actions, no gestures carrying meaning. A product about irreversible money movement does not hide actions behind discovery.
- **The signing handoff is a designed flow, not a redirect.** Before leaving for the wallet app: state what will be signed and its amount. On return: resolve to a definite state — succeeded, rejected, or failed — and never leave the member on an ambiguous screen. If the member returns before the chain confirms, show pending with the action named.
- **Pull-to-refresh on Room home and Underwriter room only.** Chain state is authoritative and the member must be able to force a re-read.
- **One primary action per surface.** Where a second action exists it is secondary-styled and navigational.
- **Banned:** confirmation dialogs on top of commit screens, toasts carrying consequential information, countdown timers as the sole expression of a deadline, carousels, onboarding tours, celebratory animation on payout, badge counts, streaks, re-engagement prompts of any kind.

## Accessibility Floor

Behavioral. Visual contrast is `DESIGN.md`'s responsibility.

- **Never color alone.** Every money state carries its meaning in text. This is the highest-priority rule in this section — the four states are paid, pending, in grace, and defaulted, and confusing any two costs someone money.
- **Screen readers:** every interactive element labeled with role and state. The grace banner is an `aria-live="assertive"` region; state changes on Room home announce. Amounts are announced as currency, not as digit strings.
- **Tap targets ≥ 48dp**, with the primary funding action at 52px minimum height.
- **Text scales with browser font-size settings to 200%** without truncating any amount, date, or control label. The schedule is the hardest case and is the one to test.
- **Focus traversal follows reading order** on every surface; the commit control is always last in order, after the disclosure it depends on.
- **No motion carries meaning.** Nothing in the product requires perceiving an animation, so `prefers-reduced-motion` degrades to no visible change.
- **Contrast floor is WCAG AA at minimum**, and the FX disclosure and grace banner are held to AAA body text — they are the two strings a member most needs to read and most likely reads in poor conditions. Measured ratios for every load-bearing pair are recorded in `DESIGN.md § Colors`; the commitment in this line is unverifiable without them, and the reviewer gate of 2026-08-12 found a grace colour that failed AA precisely because no numbers were on record.
- **Handles must survive a bad screen.** The code excludes `0`/`O` and `1`/`I`, is grouped rather than run together, and is set in tabular figures — a member comparing two handles is comparing characters, not reading a word. Screen readers announce it character by character, never as an attempted word.
- **Everything in this section is hand-built** (see § Foundation). No accessible component library underpins it, so each requirement here needs its own verification rather than being assumed from a primitive's behavior.

## Responsive & Platform

Mobile-first, single column at every width. Content max-width caps around 560px on larger viewports; the layout gains margin, never a second column.

- **Android Chrome** is the primary target and the assumed pilot device profile.
- **iOS Safari** is supported. PWA install is offered but never required, and no functionality is gated behind installing to the home screen.
- **Underwriter surfaces on a wide viewport** get more rows visible and wider tables. Exposure and applicants read well at desktop width; nothing about them is desktop-only.
- **Wallet round trip** must survive backgrounding on both platforms — iOS Safari aggressively discards background tabs, so pending transaction state is reconstructed from chain state on return, never held only in memory.

## Key Flows

Protagonists and framing mirror the PRD's UJ-1…UJ-4 verbatim.

### Flow 1 — Josel joins a paluwagan he can't be burned by (UJ-1)

1. Josel opens the invitation link on his phone. **Welcome** states what Pal3 is in four lines.
2. He connects his wallet through WalletConnect. Returns to the app with an address.
3. No attestation exists → **Verify identity** explains that a licensed provider holds his documents and Pal3 never sees them. He completes ID, employment record, and bank details in the provider's flow.
4. He returns to **Verification status**: verified at full tier, starting trust score 400, and what that score buys — a stake of 1.6× one contribution.
5. **Room terms** shows everything on one screen: $18.00 weekly, 8 rounds, payout position 6, his payout date, his stake as multiple and amount, the underwriter's fee, and the default waterfall in plain language.
6. He checks the FX acknowledgement. The commit control enables.
7. **Funding check** confirms his balance covers stake plus first contribution.
8. **Commit** — he signs once. Stake and first contribution lock.
9. **Room filling** — 6 of 8 joined, 9 days left in the open window, everything returned in full if it doesn't fill.
10. **Climax:** the room fills and starts. **Room home** shows all eight rounds, every payout date including his own — a date on a screen that nothing in the product can change.

*Edge:* room doesn't fill → cancelled state, everything returned, transaction link shown.
*Edge:* insufficient balance at step 7 → exact shortfall in dollars, commit disabled with the blocker in text.

### Flow 2 — Marivic opens a Room and picks who's in it (UJ-2)

1. Marivic connects; the app resolves her as an underwriter and reveals the **Underwrite** tab.
2. **Create room** — contribution, weekly cadence, member count, bounded by her tier (FR-18). Maximum exposure is computed and shown before she commits (FR-17).
3. **Post backstop** — the required amount is stated as a formula and a figure. She signs; it locks and cannot be withdrawn while the room runs (FR-19).
4. Admission opens. **Applicants** lists each request by handle with trust score, cycles completed, defaults, and account age — and nothing else. No name, no documents, no employer (FR-5).
5. She rejects one applicant with two prior defaults; their funds return in full.
6. **Climax:** the room reaches count and starts; her per-contribution fee begins accruing.
7. **Underwriter room** — live exposure, maximum exposure, contributions received, and any open grace window.

### Flow 3 — Dennis misses a payment and nobody else loses money (UJ-3)

1. Round 4 deadline passes with Dennis unpaid.
2. The contract opens a 48-hour grace window. **Room home** carries a persistent grace banner: hours remaining, the amount due, and exactly what happens at expiry — $18.00 slashed from his stake, 150 points off his score.
3. Every other member sees the room as "awaiting contribution" with time remaining. No blame, no name-calling in the copy.
4. Dennis pays 20 hours later through the standard contribute flow.
5. His contribution is accepted and recorded as cured: +2, not −150. The banner clears.
6. **Climax:** the scheduled recipient is paid in full, on time. Their Room home shows the payout and nothing about the near-miss.

*Edge:* no payment within 48 hours → slash executes; **Round detail** states plainly who missed, what covered it, and that the recipient was paid in full; Dennis's trust surface shows a permanent default entry with "This stays on your record."
*Edge:* stake exhausted → removal state, fully disclosed, no path back in V1.
*Known gap:* in-app-only notification means step 2 may never reach Dennis. See § Notification & Reach.

### Flow 4 — Josel gets paid, and gets something he can keep (UJ-4)

1. Round 6 boundary. The contract releases the full pot.
2. **Room home** states it plainly: $144.00 received, the date, a transaction link. No animation, no celebration.
3. He keeps contributing rounds 7 and 8 through the same flow.
4. **Cycle summary** at close: stake returned in full, +50 for a completed cycle.
5. **Climax:** his **Trust score** surface reads 460, and beneath it the consequence in the only terms that matter — his next room's stake is 1.54× instead of 1.6×. The first time saving reliably has ever earned him anything he can carry.
