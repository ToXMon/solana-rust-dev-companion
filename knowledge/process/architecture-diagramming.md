# Architecture Diagramming & Atomic Requirements

Distilled from the **2026-09-28 enrichment class** (`video/transcripts/Builders Cohort Q3 2026 - Arch Diagram - 2026_09_28 14_56 WEST - Recording.txt`) — the instructor's professional dev-shop methodology for turning an LOI/use case into a buildable design. This is the process `/solana-architect` enforces.

---

## Core principle

> "We should stop calling this an architectural diagram. Start calling it the requirement contract." (`2026_09_28`, ~01:17:50)

**Requirements come first. You do not draw anything until the requirements are written.** The diagram is just a visualization of a numbered, atomic requirement list. Every cohort, people start drawing boxes and arrows — that is the #1 mistake.

## The process (in order)

### Step 1 — Write atomic requirements, numbered

- Write requirements as `"The protocol shall …"` statements. Each statement is **numbered**.
- **Atomic = one action.** A requirement corresponds to a single state change, a single account change, or a single CPI call. If a statement contains two actions ("deposit and stake" where both are real actions), split it.
  - ❌ "The protocol allows the user to deposit and receive savings returns." — two actions, not atomic.
  - ✅ "The protocol shall allow the user to deposit tokens from a whitelisted token address." — one action.
  - ✅ "The protocol shall use [oracle] to mitigate sanctioned funds being deposited." — separate, linked requirement.
- Requirements are **human-readable** — a non-coder should understand them. That's what makes them testable for correctness before code exists.
- **Granularity test:** "employer needs to pay employees" is not atomic — *which* employees, *which* token (USDC? SOL? — it determines which ATA is invoked), how often, who selects payment type? Keep drilling until each statement implies exactly one on-chain action.
- Triggers that aren't quantifiable on-chain ("user dies") must be decomposed into external dependencies (check-in oracle, multisig sign-off) plus a requirement like "the program shall verify the trigger condition."

### Step 2 — Identify actors and signers

- Enumerate every actor explicitly: **direct actors** (depositor, staker), **beneficiaries**, **administrators**, **third-party/trusted actors** (oracle, multisig, custodian), **stakeholders** (treasury, DAO — mandatory for treasury/DAO designs).
- For each actor: define the role, where they show up in the flow, and **whether they sign**. Make all signers explicit — a sign-off referenced in one diagram but ambiguous elsewhere is a fail condition.

### Step 3 — Map state ownership

- For every state change, state whether it writes to a **program-owned account (PDA)** or an off-chain index.
- Show the **seed structure** for each PDA in the diagram (e.g., per-user vs. global state). Don't leave seed design implicit.

### Step 4 — Build the reference table

- Consolidate every account, PDA, and instruction into **one reference table** that traces 1:1 back to the numbered requirements/use cases — including external third-party programs.

### Step 5 — Draw the diagram(s)

- **Bidirectional traceability is the acceptance criterion:** a reviewer must be able to go from any requirement number to its diagram location, and from any diagram element back to its requirement. An unnumbered requirement or an unmapped diagram element = fail.
- **One diagram per state change.** Two diagrams describing the same state change under different triggers is wrong — one diagram, note the triggers.
- **One-sentence rule:** you should be able to say in one sentence which single instruction handler owns each state transition. If you can't, competing designs are tangled in one diagram — split them. This surfaces integration bugs early.
- **Consistent visual conventions:** arrows keep a consistent direction for program interactions; validation points and external dependencies get distinct boundaries; labeling/naming is identical across the overview diagram and all sub-diagrams.

### Step 6 — Cut to MVP scope

- This is a **PoC with ~2 weeks to ship and test on devnet**, not a production protocol. For every requirement ask: is this part of the MVP? If not, cut it. "The last thing you want is a 20-page requirement/diagram stack when you start coding." If your protocol is much more complex than the stablecoin-savings reference example, dial it back.

### Step 7 — Self-check, then AI red-team

1. Self-check first: requirements atomic? signers explicit? instructions clear? every element traced?
2. Then run an AI red-team pass.
3. **Treat every AI output as a finding to evaluate, not a suggestion to adopt.** Compare notes, decide deliberately.
4. Each team member writes an **individual reflection** logging any override decisions and the reason.

### Step 8 — Iterate as a living draft

- Treat the diagram like a **first draft, not a contract set in stone**. During week 1 of building, some things will prove infeasible; compliance/external blockers may not even be controllable. Expect changes — "give it real thought, but write it like a first draft; when tweaking, think of it as a living doc."
- Team workflow: **each member independently drafts the core requirements, then converge and compare** before linking them to state changes/accounts/contract calls.

## Why it pays off (instructor's receipts)

- His dev shop spent **60–70% of the first 2–3 weeks of every client engagement** on requirement iteration (multiple calls, sign-off per requirement). Result: ~1 week of coding, 15-min weekly syncs, full end-to-end mainnet delivery in under 6 weeks, auditors in by week 5, done by week 7. "There's a method to the madness."
- An instructor who was a skeptic put one week into an architecture diagram and completed the program in a weekend.
- **AI relevance:** a properly granular diagram is what you hand to a coding agent so it builds correctly — and so *you* can audit what the agent produced. "If you don't understand granularity of accounts, states, contract calls, and atomic requirements, you won't be able to sift through what the agent did." Becoming standard industry practice.
- **Team relevance:** requirements let teammates divide duties and merge code that actually connects. Going from 1 → 2 people multiplies conflicts without a shared spec.
- If finding this hard is discouraging: "if this is hard for you, how are you coding?" — the difficulty *is* the point; better here than in code.

## Getting unstuck

- **Start anywhere — words count.** A Google Doc of requirements is fine; convert to a diagram later. Use whatever tool is comfortable (Miro, draw.io, or generated SVGs from a diagramming agent). "Anywhere you are is fine, but not starting isn't."
- Feeling overwhelmed looking at a polished reference diagram is normal — it's familiarity, not skill deficit.
- It's fine to ship a diagram that mirrors the reference example's structure with your own protocol plugged in.

## Reviewer checklist (use this to grade any diagram)

- [ ] Requirements written first, each numbered, each a single action/state change/CPI
- [ ] Every requirement traces to a diagram element AND every diagram element traces to a requirement
- [ ] All actors enumerated (direct, beneficiary, admin, third-party, stakeholder) with signer status explicit
- [ ] Each state change names one owning instruction handler (one sentence)
- [ ] State ownership marked: PDA vs off-chain; PDA seed structures shown
- [ ] One reference table: accounts, PDAs, instructions, external programs → requirements
- [ ] Consistent arrow direction, distinct boundaries for validation/external deps, consistent labels across overview + sub-diagrams
- [ ] Non-MVP requirements cut
- [ ] Self-check done → AI red-team findings evaluated → override decisions logged per team member
