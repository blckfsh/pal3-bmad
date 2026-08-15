# Cowork prompt — SCF #45 supporting PDFs

Paste the block below into a Cowork session opened on `D:\programming\pal3\pal3-bmad`.

---

I need you to produce three print-ready PDFs for a Stellar Community Fund #45 grant
application for a project called Pal3 (paluwagan3). Everything you need is already
in this repository — do not invent, estimate, or infer any fact that is not in
these files.

## Source documents (read all of these first)

Planning artifacts:
- `_bmad-output/planning-artifacts/briefs/brief-paluwagan3-2026-08-08/brief.md`
- `_bmad-output/planning-artifacts/briefs/brief-paluwagan3-2026-08-08/addendum.md`
- `_bmad-output/planning-artifacts/prds/prd-paluwagan3-2026-08-08/prd.md`
- `_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/ARCHITECTURE-SPINE.md`
- `_bmad-output/planning-artifacts/architecture/architecture-paluwagan3-2026-08-08/SCF-architecture-outline.md`
- `_bmad-output/planning-artifacts/epics.md`
- `_bmad-output/planning-artifacts/implementation-readiness-report-2026-08-12.md`
- `_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/DESIGN.md`
- `_bmad-output/planning-artifacts/ux-designs/ux-paluwagan3-2026-08-11/EXPERIENCE.md`

Implementation + test state:
- `_bmad-output/implementation-artifacts/sprint-status.yaml`
- `_bmad-output/test-artifacts/test-design-architecture.md`
- `_bmad-output/test-artifacts/test-design-qa.md`

Do NOT read or quote any `.memlog.md` file — those are internal working logs.

## Hard rules

1. **No fabrication.** Every number, date, name, and claim must be traceable to a
   source document. If a fact is not there, it does not go in the PDF.
2. **Omit empty categories rather than padding them.** If there is no data for a
   requested bullet, either leave it out or state plainly that it does not exist
   yet — never write filler to occupy the heading.
3. **Do not describe anything unbuilt as if it were built.** Use future/planned
   tense for everything not yet implemented. This is a grant application; a
   reviewer catching one overstated claim discredits the whole document.
4. **Do not mention AI or AI-assisted development anywhere.**
5. Verify current build state against `sprint-status.yaml` before writing anything
   about progress. Check whether any implementation code exists in the repo
   (outside `_bmad`, `_bmad-output`, `docs`, `design-artifacts`) and let that
   determine what you claim.
6. Keep `[ASSUMPTION: ...]` items out of the PDFs, or clearly label them as
   assumptions if they are load-bearing.

---

## PDF 1 — `Pal3-Project-Description.pdf`

Purpose: the application's opening section. A reviewer who reads only this should
understand what Pal3 is, what breaks today, and what the build fixes.

The form field says "briefly describe", so this is the **shortest** of the three —
target 2 pages, and do not pad it to fill space. Primary source is `brief.md`
(Executive Summary, The Problem, The Solution, The Trust Model, The Competitive
Picture, Market Opportunity). Do not restate the whole brief; compress it.

Structure:

**1. What Pal3 is** — one tight paragraph. An on-chain, stablecoin-powered version
of the Filipino rotating savings circle (paluwagan / ROSCA), rebuilt so the
failures that kill one cannot occur. Explain the mechanism in a sentence for a
reader who has never heard of a ROSCA: a group pools money on a fixed cadence and
each member takes the full pot in turn.

**2. The problem** — trust breaks in three places. Cover member default, organizer
fraud, and the absence of any safety net or reputational record. Use the concrete
evidence in the brief: the Repa Paluwagan collapse figure and the SEC advisories
against "Online Paluwagan" schemes. Close on the framing already in the brief —
a financial tool well suited to the capital-poor is throttled because it is only
safe with people you already trust completely.

**3. The solution** — the smart contract *is* the organizer. Walk the member path
(join a room, complete KYC, lock a stake, contribute on cadence), what the contract
does (custodies every contribution, enforces the schedule, releases the full pot
each round, ordering set by trust score with a deterministic auditable tiebreak),
and automatic default handling (slash the stake first, then the underwriter
backstop, so the scheduled recipient is paid in full in every branch). State
plainly that no human ever touches the pool.

**4. The trust engine** — the four identity-anchored layers, briefly. Then the two
properties the brief calls out, because they are the strongest points in the
document: organizer fraud becomes structurally impossible rather than merely
unlikely, and trust-ranked payout inverts the core risk by paying the most-proven
members earliest, concentrating underwriter exposure where uncertainty is lowest.

**5. Why this has not been solved already** — condense the competitive table from
the brief. Two findings carry it: the regulated incumbents (GSave, Maya Savings,
Tonik Group Stash) do not rotate, so they never deliver the lump sum that makes a
paluwagan worth joining; and the earlier on-chain ROSCAs rotated but had no answer
for default, because under pseudonymous membership a defaulter's only cost is an
abandoned wallet. Include the comparison table if it fits cleanly on one page.

**6. Who it serves** — keep the three market figures separate exactly as the brief
does, and do not merge them: the serviceable cohort today (formally employed wage
and salary workers, since V1 requires employment records and bank details), the
larger unbanked population the vision reaches and V1 explicitly does not, and the
scale of the pain being priced. If the brief marks a figure `[ASSUMPTION]`, either
leave it out or label it as an estimate — do not present it as established.

**7. Scope of V1** — one short closing paragraph: V1 proves the trust engine as a
controlled pilot on Stellar/Soroban with a recruited, informed-consent cohort, and
migration to a regulated peso-stablecoin rail is a stated Phase 2 milestone. Be
explicit that this is a pilot and not a public launch.

Additionally, save a plain-text version as
`_bmad-output/planning-artifacts/scf-45/project-description.txt`, trimmed to roughly
250-400 words, so it can be pasted directly into the application form field. Same
facts, no headings, no markdown.

---

## PDF 2 — `Pal3-Current-Traction.pdf`

Purpose: show a reviewer where the project genuinely stands today.

Before writing, confirm this against the sources — I believe it to be the case,
but verify rather than trust me:

- No users or developers onboarded yet
- No on-chain transactions
- No partnerships or integrations
- No testnet or mainnet deployment (testnet deployment is story 5.2, still backlog)
- No previous grants, awards, or hackathon results — SCF #45 is the first application
- No community or social media metrics recorded

If all of that holds, do NOT write six near-empty sections. Instead structure the
document around the traction that does exist, and be direct about the rest:

**1. Where the project stands** — one honest paragraph: pre-deployment, with a
complete and reviewed build specification, entering implementation.

**2. Specification and design completeness** — the substantive traction. Cover:
- PRD, finalized
- Architecture spine — count the architectural decisions and name a few of the
  load-bearing ones; note it passed adversarial-lens and verification-lens review
- UX design — DESIGN.md and EXPERIENCE.md, both final, plus the commit-screen mockup
- Epic and story breakdown — give the actual epic and story counts from `epics.md`
- Implementation readiness assessment — state the readiness verdict, how many
  issues were found, and that they were remediated
- System-level test design completed before implementation
Present this as a table where it helps.

**3. Implementation status** — read from `sprint-status.yaml`: which epic is active,
which stories are ready for development, what remains in backlog. State the story
completion count plainly (including if it is zero).

**4. Team** — solo founder; four years of professional smart contract development
in Solidity/EVM; migrating to Rust/Soroban for this build. Use the framing already
in `SCF-architecture-outline.md` section on team: contract engineering under
adversarial conditions is the transferable capability, and a proven EVM developer
moving to Soroban is itself ecosystem value. Do not oversell; do not apologize.

**5. Pilot design** — the recruited cohort: size, cadence, cycle length, how the
underwriter side is covered, and the informed-consent framing. Make clear this is
a designed experiment that has not yet run.

**6. What does not exist yet** — a short, plain table listing: users, on-chain
activity, deployments, partnerships, prior funding, community metrics — each marked
as not yet, with the milestone that produces it where the roadmap names one.
This section is deliberate. Ending on a candid gap table reads as rigor.

---

## PDF 3 — `Pal3-Planned-Stellar-Integration.pdf`

Purpose: explain precisely how Pal3 uses Stellar. This is the stronger document —
the architecture is fully specified, so be concrete and technical.

**1. Why Stellar specifically** — draw from `SCF-architecture-outline.md` §2 and the
brief. Lead with the fee argument: the weekly contribution amount, what a
high-fee chain would cost as a percentage of it, and the fact that this appears in
the specification as a hard non-functional requirement rather than an optimization.
Then the mission alignment, and the anchor network as the Phase 2 route to a
regulated peso stablecoin.

**2. Soroban smart contracts** — the core of the document. Cover:
- The three contracts (Registry, Factory, Room) and what each owns
- The ledger-authoritative principle: contracts hold all authoritative state and
  all value; no off-chain component is ever consulted to decide an outcome that
  moves money
- `require_auth` on the invoking account as the authorization model
- Single-party authorization by design — entry points structured so the authorized
  address is the transaction source, so no detached auth-entry signing is needed,
  which keeps the product on the mainstream Stellar wallet path
- Storage durability and TTL handling, and the rule that expiry is never control
  flow — TTL is extendable by anyone, so deadlines live in room state and are
  compared against ledger timestamp via explicit invocation
- Deterministic payout ordering, and the decision to design out the need for
  verifiable randomness rather than import a cross-chain relayer dependency.
  This is a strong point — Soroban has no native VRF, and the response was to
  remove the requirement. Give it its own emphasis.
- Trust score snapshotting into the Room at join, with the join transaction
  reverting if the Registry value no longer matches what the member was shown

**3. Assets and value flow** — how contributions, stakes, and the underwriter
backstop are escrowed and settled, and the USD-pegged stablecoin position for V1.
Only describe the asset handling that the architecture actually specifies — if the
Stellar Asset Contract is not named in the sources, do not claim it by name;
describe the token interaction as the documents describe it.

**4. Wallets and client integration** — Stellar Wallets Kit, WalletConnect,
`signXDR` / `signAndSubmitXDR`, and why the absence of `signAuthEntry` in the
WalletConnect module shaped the contract authorization design. Mobile-first PWA,
React + Vite + vite-plugin-pwa, fully client-side against chain and API.

**5. Developer tooling and build chain** — `stellar contract build`, generated
TypeScript bindings via `stellar contract bindings` with hand-written encoding
prohibited, soroban-sdk version, Rust target and toolchain, `@stellar/stellar-sdk`.

**6. Anchors and the Phase 2 peso rail** — the anchor network as the realistic path
to settling in a regulated Philippine peso stablecoin, framed explicitly as a
Phase 2 milestone rather than a V1 capability.

**7. Network path and open items** — Testnet then Mainnet, gated by the Epic 5
readiness stories. Then list the genuinely open technical questions the
architecture flags for resolution before the mainnet pilot, such as the stalled-room
TTL rescue entry point and indexer/API hosting. Naming known-open items is a
credibility signal — include them.

**8. Ecosystem contribution** — all Soroban contract source published under
Apache 2.0 from first deployment, and the reasoning: for a product asking people to
escrow money alongside strangers, publicly auditable source is part of the trust
proposition rather than a concession to it.

---

## Format for all three PDFs

- Title page: document title, "Pal3 (paluwagan3)", "Stellar Community Fund #45",
  and the date
- Body in a clean serif or professional sans, roughly 11pt, generous margins,
  page numbers, and a short header carrying the project name
- Use tables for anything comparative or enumerable; a reviewer skims
- Reproduce the architecture diagram from `SCF-architecture-outline.md` in the
  Stellar Integration PDF if you can render it cleanly — if the rendering is not
  clean, omit it rather than shipping a broken diagram
- Target 2 pages for the Project Description and 3-5 pages each for the other two.
  Dense and specific beats long and padded.
- Write in plain declarative English. No marketing voice, no superlatives, no
  "revolutionary" or "cutting-edge". The specification's own register is the right
  one — carry it over.

Save all three PDFs to `_bmad-output/planning-artifacts/scf-45/`.

When you are done, give me a short list of every place where you found no
supporting data and therefore left something out, so I can decide whether to
supply that information myself.
