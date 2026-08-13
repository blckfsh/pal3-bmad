# Spine Pair Review — paluwagan3

**Run:** 2026-08-12, update-pass reviewer gate.
**Method note:** subagents unavailable in this session, so this lens ran inline rather than in an independent context — a weaker check than the gate specifies. Contrast values below were computed, not estimated.

## Overall verdict

The pair is a strong downstream contract: every token reference resolves, flows mirror the PRD's journeys verbatim, and the invented sections (Money Legibility, Irreversibility & Commitment, Trust Score Legibility, Notification & Reach) each carry decisions no standard section would hold. One critical defect undercuts it — the grace state color fails WCAG AA as text everywhere it is used, while EXPERIENCE.md holds the grace banner to AAA. The two spines contradict each other on the single highest-stakes element in the product.

## 1. Flow coverage — strong

Checked: `sources` frontmatter → PRD UJ-1…UJ-4. All four have Key Flows with named protagonists (Josel, Marivic, Dennis, Josel), numbered steps, explicit climax beats, and edge paths.

### Findings
- **medium** FR-16 removal-after-stake-exhaustion has a State Patterns row and a terminal treatment but no flow of its own; it is the far edge of Flow 3 (§ Key Flows). It is also the most harmful outcome a member can experience — payout forfeited, no path back. *Fix:* either a short fifth flow or an explicit edge branch under Flow 3 walking the removal from the member's side. Already self-flagged in § IA closure note, so this is a known gap rather than a miss.

## 2. Token completeness — broken

Checked: every YAML token in `DESIGN.md` frontmatter, and every `{path.to.token}` reference in `EXPERIENCE.md` (`{colors.state-grace-wash}`, `{spacing.commit-gap}`, `{typography.meta}`). All three resolve. All colors carry hex; light-only by decision, so no dark pairs are owed.

### Findings
- **critical** `state-grace` (`#A5701A`) fails WCAG AA for body text against every surface it is used on: **3.86:1** on `state-grace-wash`, **4.26:1** on `surface-raised`, **3.93:1** on `surface-base` (AA requires 4.5:1). Meanwhile `EXPERIENCE.md § Accessibility Floor` states the grace banner is *"held to AAA body text"* (7:1). The two spines directly contradict each other, and they do it on the grace banner — which the spine itself calls "the highest-stakes state in the product," carrying the hours remaining and the exact slash consequence, read by a member in poor conditions. The "in grace" status pill text fails equally. *Fix:* darken to `#6B4710`, which measures **7.51:1** on the wash and **8.29:1** on raised — AAA on both, and still unmistakably amber rather than red.
- **medium** No contrast values are stated anywhere for load-bearing pairs. The commitments are qualitative ("AA at minimum", "AAA body text") with nothing to check them against, which is how the defect above survived to `final`. *Fix:* record measured ratios for the load-bearing combinations in `DESIGN.md § Colors`.
- **low** `ink-disabled` on `surface-base` is **2.44:1**. WCAG exempts disabled controls and `DESIGN.md` already restricts this token to genuinely unavailable controls, so this is conformant — but it is worth stating explicitly so no one reaches for it as a de-emphasis grey.

Verified passing: `ink-primary` on base **16.42:1**; `accent` on raised **9.35:1**; `ink-inverse` on accent **9.35:1**; `state-default` on its wash **7.02:1** (AAA); `state-paid` on raised **6.30:1** (AA); `ink-secondary` on base **5.86:1** (AA).

## 3. Component coverage — adequate

Checked: 10 components in `DESIGN.md` frontmatter and body against 10 rows in `EXPERIENCE.md § Component Patterns`.

### Findings
- **medium** `button-secondary` has a visual spec in `DESIGN.md` but no behavioral row in `EXPERIENCE.md`. `DESIGN.md` says it is used for "not now" and for navigating away from a commitment — behavioral rules with no home in the spine that owns behavior. *Fix:* add a Component Patterns row.
- **medium** Component names diverge across the pair: `DESIGN.md` calls it **Button (primary)**, `EXPERIENCE.md` calls it **Primary action**. A downstream consumer extracting component names gets two lists. *Fix:* align on one name in both files.
- **low** **Signing handoff** has a behavioral row but no `DESIGN.md` visual spec. Defensible — it is a flow, not a surface element, and the row points to § Interaction Primitives. No action.

## 4. State coverage — thin

Checked: all 20 IA surfaces against the 19 rows in § State Patterns. Error, offline, pending, expired-session, and every money state are covered thoroughly — this is the strongest part of the spine. Cold and empty states are the gap.

### Findings
- **high** Three surfaces have no empty or cold state specified: **Activity** (a member who has taken no on-chain action yet), **Applicants** (an underwriter whose room has opened admission but attracted no requests), and **Trust score** (a newly verified member whose history is empty — which is *every* member at pilot start, and the surface SM-5 measures). *Fix:* three State Patterns rows.
- **low** **Members** has no pre-start state — before a room fills, the roster is partial by definition. Arguably covered by Room filling. No action needed if that is the intent; worth one clause.

## 5. Visual reference coverage — strong

`mockups/room-terms-commit.html` is linked inline from `EXPERIENCE.md § IA` and `§ Irreversibility & Commitment`, and from `DESIGN.md § Components`, each naming what it illustrates. Spines-win-on-conflict is stated in both files. `imports/` is empty by fact, not omission. No orphans.

### Findings
- **medium** The mock states `rate as of 11 Aug, 9:41` — a timestamp with no named source, which satisfies half of what AD-16 and § Money Legibility require. Not corrected in this run because the FX provider is not selected and naming one would be invention. *Fix:* two strings, once the provider is chosen. Logged in the memlog as a gap.

## 6. Bloat & overspecification — strong

`DESIGN.md` prose carries editorial voice, which the spec permits and which does real work here — the Josel framing in § Brand & Style is why the palette is warm rather than clinical. `EXPERIENCE.md` stays behavioral. Pixel values appear only where a token defines them. § Foundation restates a few PRD-fixed constraints (PWA, WalletConnect, cadence); mild duplication, but it prevents a reader treating settled constraints as open. No action.

## 7. Inheritance discipline — adequate

`sources` frontmatter resolves in both files and now correctly binds the architecture spine. UJ names and protagonist names are verbatim from the PRD. `{token}` references all resolve.

### Findings
- **medium** Component naming divergence (see § 3) is the concrete instance of an inheritance-discipline miss.
- **low** The spines use lowercase domain nouns in UI prose ("room", "member", "stake") where the PRD Glossary capitalizes them. Correct for microcopy — `ARCHITECTURE-SPINE.md` requires verbatim Glossary terms in *code*, not in member-facing copy. No action.

## 8. Shape fit — strong

`DESIGN.md` sections are in canonical order with none out of place. `EXPERIENCE.md` carries all eight required defaults plus Responsive & Platform (triggered by the mobile/desktop split). Inspiration & Anti-patterns is omitted, defensibly and with the reason logged — the user supplied no reference products, and inventing them would be authoring. The four invented sections each hold decisions no default section would.

## Mechanical notes

- Frontmatter complete in both files; `status: final`, `updated: 2026-08-12` on both.
- No broken cross-references; the two inline mock links resolve.
- No Mermaid in either spine.
- `typography.handle` added this pass is defined and referenced from `components.member-status-row`.
- Component count reconciles at 10 per file; the mismatch is naming and one missing behavioral row, not a missing component.
