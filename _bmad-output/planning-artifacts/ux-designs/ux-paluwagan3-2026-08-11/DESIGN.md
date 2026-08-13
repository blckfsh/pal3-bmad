---
name: Pal3
description: Mobile-first savings-circle app for a Philippine pilot cohort. Calm, institutional, light-only. Money is legible before it is beautiful.
status: final
created: 2026-08-11
updated: 2026-08-12
sources:
  - "{planning_artifacts}/prds/prd-paluwagan3-2026-08-08/prd.md"
  - "{planning_artifacts}/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md"
colors:
  surface-base: '#F6F6F3'
  surface-raised: '#FFFFFF'
  surface-sunken: '#EDEDE8'
  ink-primary: '#16181C'
  ink-secondary: '#5A6068'
  ink-disabled: '#9AA0A7'
  ink-inverse: '#FFFFFF'
  accent: '#124F4B'
  accent-pressed: '#0D3B38'
  border-hairline: '#E2E3DE'
  border-strong: '#C7C9C2'
  state-paid: '#2E6B4F'
  state-grace: '#6B4710'
  state-default: '#96272B'
  state-grace-wash: '#FBF3E2'
  state-default-wash: '#FAEDED'
typography:
  display:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '32px'
    fontWeight: '600'
    lineHeight: '38px'
    letterSpacing: '-0.02em'
  title:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '22px'
    fontWeight: '600'
    lineHeight: '28px'
    letterSpacing: '-0.01em'
  body:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '16px'
    fontWeight: '400'
    lineHeight: '24px'
  body-strong:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '16px'
    fontWeight: '600'
    lineHeight: '24px'
  meta:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '13px'
    fontWeight: '400'
    lineHeight: '18px'
  amount:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '28px'
    fontWeight: '600'
    lineHeight: '34px'
    letterSpacing: '-0.01em'
    fontVariantNumeric: 'tabular-nums'
  amount-inline:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '16px'
    fontWeight: '600'
    lineHeight: '24px'
    fontVariantNumeric: 'tabular-nums'
  handle:
    fontFamily: "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif"
    fontSize: '16px'
    fontWeight: '500'
    lineHeight: '24px'
    letterSpacing: '0.04em'
    fontVariantNumeric: 'tabular-nums'
rounded:
  DEFAULT: '8px'
  sm: '6px'
  md: '10px'
  lg: '14px'
  full: '9999px'
spacing:
  '1': '4px'
  '2': '8px'
  '3': '12px'
  '4': '16px'
  '5': '24px'
  '6': '32px'
  '7': '48px'
  '8': '64px'
  margin-mobile: '16px'
  gutter: '12px'
  commit-gap: '32px'
components:
  button-primary:
    background: '{colors.accent}'
    backgroundPressed: '{colors.accent-pressed}'
    text: '{colors.ink-inverse}'
    typography: '{typography.body-strong}'
    radius: '{rounded.md}'
    minHeight: '52px'
    paddingX: '{spacing.5}'
  button-secondary:
    background: 'transparent'
    text: '{colors.ink-primary}'
    border: '1px solid {colors.border-strong}'
    typography: '{typography.body-strong}'
    radius: '{rounded.md}'
    minHeight: '52px'
  amount-display:
    typography: '{typography.amount}'
    color: '{colors.ink-primary}'
    secondaryTypography: '{typography.meta}'
    secondaryColor: '{colors.ink-secondary}'
  status-pill:
    typography: '{typography.meta}'
    radius: '{rounded.full}'
    paddingX: '{spacing.2}'
    paddingY: '{spacing.1}'
    border: '1px solid currentColor'
    background: 'transparent'
  terms-row:
    labelTypography: '{typography.body}'
    labelColor: '{colors.ink-secondary}'
    valueTypography: '{typography.amount-inline}'
    valueColor: '{colors.ink-primary}'
    divider: '1px solid {colors.border-hairline}'
    paddingY: '{spacing.3}'
  schedule-row:
    background: '{colors.surface-raised}'
    radius: '{rounded.sm}'
    paddingY: '{spacing.3}'
    paddingX: '{spacing.4}'
    markerWidth: '3px'
  member-status-row:
    handleTypography: '{typography.handle}'
    statusTypography: '{typography.meta}'
    paddingY: '{spacing.3}'
    divider: '1px solid {colors.border-hairline}'
  trust-score-panel:
    background: '{colors.surface-raised}'
    radius: '{rounded.lg}'
    scoreTypography: '{typography.display}'
    trackColor: '{colors.surface-sunken}'
    fillColor: '{colors.accent}'
    trackHeight: '6px'
    trackRadius: '{rounded.full}'
  notice-banner:
    radius: '{rounded.md}'
    paddingX: '{spacing.4}'
    paddingY: '{spacing.3}'
    titleTypography: '{typography.body-strong}'
    bodyTypography: '{typography.meta}'
    borderWidth: '1px'
  disclosure-block:
    background: '{colors.surface-sunken}'
    radius: '{rounded.md}'
    typography: '{typography.meta}'
    color: '{colors.ink-primary}'
    paddingX: '{spacing.4}'
    paddingY: '{spacing.4}'
---

# Pal3 — Design Spine

> Visual identity. Behavior, IA, states, and flows live in `EXPERIENCE.md`. Where a mock, wireframe, or import disagrees with this file, this file wins.

## Brand & Style

Pal3 is asking a specific person for a specific act of faith: Josel, who lost ₱18,000 to a paluwagan two years ago, is being asked to lock eight weeks of savings into software. Nothing in the visual language may read as a pitch. The product's argument is that the machine cannot betray him — so the interface should look like something that will still be running, unchanged, when his eighth round settles.

That produces an institutional posture rather than a fintech one. Sparse surfaces, hairline structure, no gradients, no illustration, no celebratory motion, no marketing voice inside the product. Color appears where money changes state and essentially nowhere else. The most visually prominent thing on any screen is a number the member can act on.

It is institutional but not cold, and the difference is carried by warmth in the neutrals rather than by decoration. The base surface is a warm off-white, not a clinical grey-blue. There are no mascots, no confetti on payout, and no progress gamification — the trust score is a financial record, not a badge.

`[ASSUMPTION]` The whole aesthetic direction below was derived from the chosen "calm and institutional" temperament plus the PRD's cohort description. No brand assets, logo, or prior visual work were supplied. Every specific value here is a first proposal, not a ratified brand.

## Colors

The palette is deliberately small: two neutral surfaces, three ink weights, one accent, three money-state colors. A member should be able to learn what color means in this product in one round.

- **Warm Paper (`#F6F6F3`)** is the app canvas. Warm enough to avoid the clinical bank-portal feeling that would remind Josel of the institution that already turned him down.
- **White (`#FFFFFF`)** is the raised surface — cards, rows, panels. Hierarchy comes from tone against Warm Paper plus a hairline, never from shadow.
- **Sunken (`#EDEDE8`)** backs disclosure blocks and inert tracks. It reads as "this is the record, not an action."
- **Ink (`#16181C` / `#5A6068` / `#9AA0A7`)** is the three-step text ramp. Amounts and member-facing decisions are always at full ink; supporting labels sit at secondary; disabled is reserved for genuinely unavailable controls, never for de-emphasis.
- **Deep Teal (`#124F4B`)** is the only chromatic brand color, and it means *commitment*: the primary action, the trust-score fill, the active round marker. It is never used for decoration, never as a background wash, and never for status.
- **Paid (`#2E6B4F`)**, **Grace (`#6B4710`)**, **Default (`#96272B`)** are money-state colors and are reserved absolutely to that meaning. Paid is muted on purpose — a member meeting their obligation is the normal case, not an achievement. Grace is amber because it is a countdown, not a failure — but a *dark* amber, because it carries the grace banner's text and that text is held to AAA. A lighter amber was measured and rejected: it failed AA on every surface it appeared on. Default is deep red rather than bright: it is a permanent record entry, and it should read as gravity rather than alarm.

**Measured contrast, load-bearing pairs.** Recorded because the commitments in `EXPERIENCE.md § Accessibility Floor` are unverifiable without numbers, and an unmeasured palette is how a failing colour reaches `final`:

| Foreground on background | Ratio | Target |
|---|---|---|
| `ink-primary` on `surface-base` | 16.42:1 | AAA |
| `accent` on `surface-raised`, and `ink-inverse` on `accent` | 9.35:1 | AAA |
| `state-grace` on `state-grace-wash` | 7.51:1 | AAA — grace banner |
| `state-grace` on `surface-raised` | 8.29:1 | AAA |
| `state-default` on `state-default-wash` | 7.02:1 | AAA |
| `state-paid` on `surface-raised` | 6.30:1 | AA |
| `ink-secondary` on `surface-base` | 5.86:1 | AA |
| `ink-disabled` on `surface-base` | 2.44:1 | exempt — genuinely disabled controls only, never de-emphasis |
- **Grace Wash (`#FBF3E2`)** and **Default Wash (`#FAEDED`)** are the only background fills in the system, used exclusively behind the grace banner and a defaulted row.

Never: teal as a status color, red for anything that is not a default or a destructive confirmation, green for anything that is not a settled contribution or payout, and any color at all on the trust-score history other than ink.

## Typography

One family — the platform system stack — at six roles. A PWA renders in a browser, so nothing here inherits from UIKit or Compose; the stack resolves to San Francisco on iOS Safari and Roboto on Android Chrome, which is the right familiarity for both.

- **`display`** — the payout amount and the trust score. At most one per screen, often zero.
- **`title`** — screen titles and section heads.
- **`body`** / **`body-strong`** — everything a member reads to make a decision. Never below 16px; this cohort includes people reading on cracked screens in poor light.
- **`meta`** — timestamps, handles, secondary peso conversions, disclosure text. Never used for an amount the member must act on.
- **`amount`** / **`amount-inline`** — every monetary figure, always with `tabular-nums`. Non-negotiable: the schedule is a column of amounts and dates, and proportional figures make a column of money unreadable at a glance.
- **`handle`** — the member code, and nothing else. Tabular figures and a little letter-spacing, because it is a reference to be compared character by character rather than read as a word. Never used for prose.

No all-caps labels, no letter-spaced small text, no decorative weights. Type scales with the browser's font-size setting; nothing is locked in `px` at the layout level.

## Layout & Spacing

Scale: 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64. Single column at every width — the underwriter surfaces gain breathing room on a wider viewport, not a second column.

Mobile margin is 16px. `commit-gap` (32px) is a named token with a rule attached: it is the minimum vertical distance between the last piece of disclosure and any control that commits funds. Nothing that moves money sits within a thumb-slip of the text explaining it.

Screens that commit funds are the exception to density — they get the most whitespace in the product. Screens that report state (schedule, member status, exposure) are the densest, because their job is to let the eye scan a whole cycle at once.

## Elevation & Depth

There is no shadow in Pal3. Raised surfaces are distinguished by tone (`{colors.surface-raised}` on `{colors.surface-base}`) and a `{colors.border-hairline}` edge. Sheets and modals get a scrim, and that is the only depth cue in the system.

Hierarchy comes from position, tone, and type weight. A product whose claim is that nothing is hidden should not use a visual language built on things floating above other things.

## Shapes

`{rounded.sm}` (6px) for status pills' square siblings, schedule rows, and inputs. `{rounded.md}` (10px) for buttons, banners, and disclosure blocks. `{rounded.lg}` (14px) for the trust-score panel — the one surface allowed to feel like an object. `{rounded.full}` for status pills and the score track only.

No circles as surfaces, no pill-shaped buttons. Corners are soft enough to feel current and square enough to feel like a record.

## Components

- **Button (primary)** — solid `{colors.accent}`, 52px minimum height, full-bleed to the mobile margin on commit screens. One per screen. Pressed state darkens to `{colors.accent-pressed}`; there is no hover state worth designing for on the primary surface.
- **Button (secondary)** — hairline outline, ink label. Used for "not now" and for navigating away from a commitment. Never styled to compete with primary.
- **Amount display** — the monetary figure in `{typography.amount}` at full ink, with an optional indicative peso line beneath in `{typography.meta}` at `{colors.ink-secondary}`, always prefixed `≈`. The dollar figure is the number of record; the peso line is a courtesy and is styled to say so. See `EXPERIENCE.md § Money Legibility` for the attribution rule that governs it.
- **Status pill** — outline only, never filled, colored by money state (`{colors.state-paid}`, `{colors.state-grace}`, `{colors.state-default}`, or `{colors.ink-secondary}` for pending). Text always carries the meaning; the color only reinforces it.
- **Terms row** — label left in `{colors.ink-secondary}`, value right in `{typography.amount-inline}` at full ink, hairline divider beneath. The room terms screen is built entirely from these; it should read like a contract, because it is one.
- **Schedule row** — one per round. Round number, date, recipient handle, amount. A 3px `{colors.accent}` marker on the leading edge of the current round; completed rounds carry a `{colors.state-paid}` pill; the member's own round is the only row set in `{typography.body-strong}`.
- **Member status row** — handle left in `{typography.handle}`, status pill right. No avatars, no names, no photos — the product does not know members' legal identities and the UI should never imply otherwise. The handle is a short alphanumeric code derived from the member's wallet address, set in two separated groups and drawn from an alphabet that excludes `0`/`O` and `1`/`I`; it reads as an account reference, not as a name the product gave someone.
- **Trust score panel** — score in `{typography.display}`, a 6px track filled to score/1000 in `{colors.accent}`, and the derived consequence stated in `{typography.body}` beneath ("Your stake is 1.6× one contribution"). The panel always shows the number, the fill, and what the number *costs or saves* — never the number alone.
- **Notice banner** — hairline border in the relevant state color over the matching wash. Two variants only: grace (amber) and default (red). No informational or success banners exist in the system.
- **Disclosure block** — sunken background, `{typography.meta}` at full ink rather than secondary. Legal and FX text is small but never greyed out; low contrast on the FX disclosure would be a genuine problem, not a stylistic one.

→ Composition reference: [`mockups/room-terms-commit.html`](mockups/room-terms-commit.html) — terms row, amount display, disclosure block, trust score panel, and primary action at 1:1. This spine wins on conflict.

## Do's and Don'ts

| Do | Don't |
|---|---|
| Reserve `{colors.accent}` for commitment and current-round marking | Use teal as a status, a background wash, or decoration |
| Use `tabular-nums` on every monetary figure | Set amounts in proportional figures anywhere |
| Carry meaning in the status text; let color reinforce | Encode any state in color alone |
| Keep `{spacing.commit-gap}` between disclosure and any funding control | Place a commit button within a thumb-slip of its explanation |
| State the trust score with its consequence | Show the score as a badge, level, streak, or grade |
| Separate surfaces with tone and a hairline | Introduce shadows, gradients, or floating cards |
| Set the FX disclosure at full ink | Grey out, collapse, or truncate the FX disclosure |
| Let payout be a plain, complete statement of fact | Celebrate payout with confetti, animation, or exclamation |
