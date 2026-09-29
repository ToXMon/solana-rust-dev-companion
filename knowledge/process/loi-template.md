# Letter of Intent (LOI) / Capstone Proposal Template

Use this template to draft the first iteration of your Builders Cohort capstone LOI. The goal is to get your ideas solid, not permanent. Treat it as a proposal you will red-team and refine.

## 1. Value Proposition

**In one sentence:** What problem are you solving and for whom?

- **Problem:** What is broken, expensive, slow, or inaccessible today?
- **Target user:** Who feels this pain most acutely?
- **Solana advantage:** Why is this specifically better on Solana than on traditional rails or another chain?

> Tip: be concrete. “Lower fees” is generic. “Sub-cent micropayments for AI agents” is specific.

## 2. Product-Market Fit

- **Minimum viable product (MVP):** What is the smallest thing you can build and validate in the cohort?
- **Validation signal:** How will you know users want this? (waitlist, pilot partner, transaction volume, etc.)
- **Killer feature:** What is the one capability that only your product offers in this form?

## 3. Founder-Market Fit

- **Why you?** What domain expertise, network, or technical edge do you bring?
- **Why now?** What market, regulatory, protocol, or ecosystem shift makes this timing right?
- **Why Solana?** Specific ecosystem tailwinds: Alpenglow finality, Token-2022, RWA growth, x402/agent payments, Metaplex Core, perp momentum, etc.

## 4. Market & Competitors

| Competitor / Alternative | What they do | Their weakness | Your differentiator |
|--------------------------|--------------|----------------|-------------------|
| (Traditional finance) | | | |
| (Crypto competitor) | | | |
| (Status quo / manual process) | | | |

- **TAM / SAM / SOM:** Rough sizing if available.
- **Trends:** Cite live data sources where possible (RWA.xyz, DefiLlama, Messari, Birdeye).

## 5. Target Markets

- **Primary market:** First beachhead — who pays first?
- **Secondary markets:** Where do you expand after proving the core loop?
- **Geography / regulatory:** Any jurisdiction or compliance constraints?

## 6. Use Cases

List 2–4 concrete scenarios with a user, action, and outcome:

1. **Use case A:** As a [user], I want to [action] so that [outcome].
2. **Use case B:** ...
3. **Use case C:** ...
4. **Use case D:** ...

## 7. Technical Approach

- **On-chain components:** programs, PDAs, tokens/NFTs, Token-2022 extensions, confidential transfers, Metaplex Core, etc.
- **Off-chain components:** oracles, KYC, custody, NAV calculation, frontend, indexer.
- **Composability:** Which existing Solana protocols do you integrate with? (Jupiter, Drift, Ondo, Pyth, Switchboard, Helius, etc.)
- **Security model:** trust assumptions, admin keys, audit plan, oracle risks.

## 8. Go-to-Market / Launch Plan

- **First 100 users:** How do you get initial traction?
- **Partnerships:** Any potential integrations or design partners?
- **Content/community:** How will you build in public?

## 9. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Regulatory | | | |
| Technical (oracle, bug) | | | |
| Market adoption | | | |
| Competitive response | | | |

## 10. Cohort Execution Plan

- **Week-by-week milestones:** What will you build each week?
- **Demo target:** What do you want to show at the end?
- **Resource needs:** mentors, RPC, testnet tokens, frontend help?

---

## How to use this with the agent system

1. Fill out each section yourself first (human-first thinking is the cohort's recommendation).
2. Run `/solana-loi` and paste your draft + the rubric.
3. The agent will red-team your assumptions, tighten the language, and suggest a realistic Solana build path.
4. Ground market claims by running `/solana-research` on the latest ecosystem data.
5. Iterate 2–3 times, then submit.
