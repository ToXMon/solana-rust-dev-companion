# Capstone, LOI & architecture diagrams — cohort notes

All cohort guidance on assignments, the Letter of Intent, capstone logistics, and the Sep 28 enrichment-week session on architecture diagrams and atomic requirements. Read this before starting any capstone deliverable. The distilled methodology lives in `knowledge/process/architecture-diagramming.md`; worked artifacts in `examples/`.

## Weekly coding assignments

- **Aug 31 assignment:** complete vault program — `initialize`, `deposit`, `withdraw`, `close`. (`2026_08_31`, ~00:01:00–00:02:40; ~01:46:31–01:49:37)
- **Sep 2 assignment:** complete escrow `make`/`take`/`refund`. Starter repo shared in Discord includes `make` and `refund` with tests. (`2026_09_02`, ~01:34:06–01:37:05)
- **Sep 4 assignment:** implement `take` and `update` instructions; optional timed escrow extension. (`2026_09_04`, ~00:03:15–00:04:06)
- **Sep 7/9 AMM assignment:** implement initialize, deposit, withdraw, swap for a constant-product AMM, with tests. Optional extension: fees and/or re-implement the curve library yourself. (`2026_09_09`, ~00:02:38–00:03:05; ~01:21:36–01:22:46)
- **Sep 21/23 NFT staking assignments:** see `nfts-metaplex-core.md`.
- **Sep 18:** no new formal assignment; action item was completing the team-formation form ASAP (mutual teammate selections). Suggested self-directed exploration: review the CPI-guard/permanent-delegate repo; try enabling/disabling CPI guard and observe which transactions succeed or fail. (`2026_09_18`, ~00:02:36–00:05:29; ~00:09:59–00:10:13)
- **Sep 14:** no new formal assignment; suggested self-directed exploration of the SPL vs Token-2022 mint byte layout and TLV format. Office hours offered. (`2026_09_14`, ~00:29:34–00:30:24; ~01:34:05–01:34:10)

## Capstone / Letter of Intent (LOI)

- **LOI is a proposal, not a permanent commitment.** "Business planning is never set in stone… when we say something is due, we mean get your ideas solid, not cement them." (`2026_09_07`, ~00:04:20–00:05:14)
- **LOI Part 1 contents:** value proposition, product-market fit, founder-market fit, competitors, target markets, use cases. (`2026_09_07`, ~00:07:27–00:12:25)
- **Recommended workflow:**
  1. Human-first thinking and mapping.
  2. Red-team your own draft (or with AI in research mode) to find holes.
  3. Refine and document why you changed things.
  4. Submit as Google Doc or PDF to Google Classroom. (`2026_09_07`, ~00:13:02–00:14:38)
- **Do not use AI for the initial thinking.** Pasting the assignment into AI upfront yields poor results; AI is best used after you have a framework. (`2026_09_07`, ~00:12:39–00:13:50)
- **Capstone domains:** real-world assets, tokenization, collectibles, payments, point-of-sale. Consider local markets outside US/Europe/major Asia. (`2026_09_07`, ~00:06:04–00:07:27)
- **Team formation:** 3–4 person teams preferred (later revised: groups of two or three, pick up to two teammates, mutual selection matters — if A names B but B does not name A, that pairing is unlikely). Friday session includes meet-and-greet/breakout rooms grouped by similar ideas. (`2026_09_07`, ~00:03:05–00:04:19; ~01:26:17–01:26:38; `2026_09_18`, ~00:02:36–00:05:29)
- **Next deliverable after LOI:** on-chain requirements / architectural diagram mapping use cases to atomic actions. (`2026_09_07`, ~00:14:54–00:15:18)
- **Capstone execution:** "moving from absorbing information to 100% applying information." Feeling overwhelmed is expected; plan, divide and conquer with teammates, use office hours, daily syncs. (`2026_09_23`, ~00:08–00:11)

## Community expectations

- Share wins, completed programs, lessons learned, or MVP recognition on X/LinkedIn. "You are always building a public record." (`2026_09_04`, ~00:05:47–00:06:42)
- Top cadets recognized weekly for consistency, initiative, helping others, and social sharing. (`2026_09_04`, ~00:05:00–00:05:46)
- "If you don't know what you're doing, share your screen first. That's when the seniors can actually help you learn." (`2026_09_09`, ~01:19:02–01:21:00)

## Architecture diagrams & atomic requirements (Sep 28 enrichment week)

**Source file:** `Builders Cohort Q3 2026 - Arch Diagram - 2026_09_28 14_56 WEST - Recording.txt` (01:20:35). ASR is noisy in places; speaker identities not assigned. Full distilled methodology lives in `knowledge/process/architecture-diagramming.md`.

The instructor shared a real client artifact (a stablecoin savings protocol) as the reference/anchor format, then ran breakout-room requirement critiques.

### Mental models

- **Requirements are the deliverable; the diagram is a visualization of them.** "We should stop calling this an architectural diagram — start calling it the requirement contract." Don't draw until requirements exist. (`2026_09_28`, ~01:08:11; ~01:17:50)
- **Atomic = one action.** Each numbered `"The protocol shall …"` statement maps to one state change, account change, or CPI call. "Deposit and stake" where both are real actions is not atomic. (`2026_09_28`, ~00:04:59–00:06:15; ~01:03:28–01:03:36)
- **Bidirectional traceability.** Number every requirement; a reviewer must trace requirement → diagram element and diagram element → requirement. Anything unmapped is a fail condition. (`2026_09_28`, ~00:19:00–00:20:01)
- **One-sentence rule.** Be able to say in one sentence which single instruction handler owns each state transition; otherwise competing designs are tangled and integration bugs surface late. (`2026_09_28`, ~00:15:15–00:16:09)
- **Actors must be explicit, including signer status.** Direct actors, beneficiaries, administrators, third-party/trusted actors (oracle/multisig), and stakeholders (mandatory for treasury/DAO designs). (`2026_09_28`, ~00:13:34–00:15:15; ~00:21:54–00:22:11)
- **Granularity example (private payroll):** "employer pays employees" → not atomic. Which employees? Paid in SOL or USDC — that determines which ATA is called? Who selects payment type? Frequency? (`2026_09_28`, ~00:53:05–00:54:16; ~01:03:46–01:04:16)
- **Non-quantifiable triggers decompose into external dependencies.** "Transfer on death" isn't enforceable on-chain; it becomes "verify the trigger" plus oracle/check-in/multisig requirements. (`2026_09_28`, ~01:09:00–01:13:18)
- **Cut to MVP.** ~2 weeks to ship a PoC to devnet — cut every requirement that isn't core; avoid a "20-page requirement stack." If the design is much more complex than the reference example, dial it back. (`2026_09_28`, ~00:10:48–00:11:29; ~00:12:47–00:13:07; ~01:19:02–01:19:15)
- **Living draft, not a contract.** Expect changes during week 1 of building (infeasible items, uncontrollable compliance blockers). Put real thought in, but don't treat v1 as final. (`2026_09_28`, ~00:11:42–00:12:39)
- **AI red-team = findings, not commands.** Self-check first, then use AI to red-team; evaluate each output deliberately; each team member logs override decisions with reasons. (`2026_09_28`, ~00:20:11–00:21:39)
- **Team workflow:** each member independently drafts core requirements, then converge and compare before linking to state changes/accounts. (`2026_09_28`, ~01:18:23–01:18:43)
- **Diagram conventions:** consistent arrow direction for program interactions; distinct boundaries for validation points/external dependencies; identical labeling across overview and sub-diagrams; one reference table tying every account/PDA/instruction/external program 1:1 to requirements; PDA seed structures visible; one diagram per state change (not two diagrams for the same change under different triggers). (`2026_09_28`, ~00:16:49–00:17:19; ~00:21:41–00:23:13)
- **Why it matters:** dev shop spent 60–70% of the first 2–3 client weeks iterating requirements → ~1 week coding, 15-min syncs, mainnet in <6 weeks, audits done by week 7. It's also what you hand to a coding agent — and what lets you audit the agent's output. (`2026_09_28`, ~00:08:50–00:09:52; ~00:33:44–00:34:56)
- **Start anywhere.** A Google Doc of requirements is fine; convert to a diagram later. "Anywhere you are is fine, but not starting isn't." (`2026_09_28`, ~00:26:29–00:26:36; ~00:32:08–00:32:15)

### Practical tips

- State ownership must be marked: does each state change write to a program-owned PDA or an off-chain index? Show PDA seed structure (global vs per-user state). (`2026_09_28`, ~00:17:23–00:18:32)
- Tools mentioned: Miro/draw.io-style boards for drafting; a student demoed an AI tool that generated architecture diagrams from a repo's escrow workspace (instructions, accounts, CPI calls, state changes). (`2026_09_28`, ~00:26:32–00:26:36; ~00:30:16–00:31:37)

### Deliverables

- **Deliverable: architecture diagram (the "requirement contract") for the capstone**, following the shared reference format. Breakout exercise: critique teammates' requirements for atomicity with instructor feedback. (`2026_09_28`, ~00:09:54–00:10:48)
- Each team member produces an individual reflection on AI red-team findings and override decisions. (`2026_09_28`, ~00:21:09–00:21:39)

### Example programs discussed

| Example | What it shows | Source segment |
|---------|---------------|----------------|
| Stablecoin savings protocol (reference artifact) | The model format: numbered atomic requirements linked to contract actions (deposit, unstake, rewards, fees to separate vaults). | `2026_09_28`, ~00:03:02–00:07:42 |
| Private payroll / stealth payments | Wholesome student design critiqued live: encryption complexity is a dependency, not the core requirement; boil to "employer sends stablecoin / recipient receives" then attach Token-2022 confidential transfers + one-time accounts as dependencies. | `2026_09_28`, ~00:50:37–00:59:28 |
| Dead-man's-switch inheritance | "Death" isn't a quantifiable trigger → decompose into beneficiary selection, trigger verification, external dependency (oracle/multisig), account transfer. | `2026_09_28`, ~01:08:57–01:13:18 |

Related: `knowledge/process/architecture-diagramming.md`, `knowledge/process/loi-template.md`, `examples/loi/`, `examples/requirements/`, `quotes.md`.
