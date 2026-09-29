# Verbatim quotes & heuristics — cohort transcripts

Every verbatim instructor quote preserved from the cohort transcripts, grouped by topic. Quotes are transcript wording (ASR, not human-verified); `[ASR corrected]` marks obvious transcription errors. Citations refer to the original recordings.

## Accounts, PDAs, and mental models

> "Rent is more like a deposit… you can reclaim it when the account closes."
> — `2026_08_24`, ~00:21:45

> "What if there was no PDA?"
> — Thought exercise used repeatedly to motivate PDAs. `2026_08_24`, ~01:43:29

> "All ATAs are PDAs, but not all PDAs are ATAs."
> — `2026_08_26`, ~00:18:42

> "Transfer moves balances without changing supply; mint increases supply; burn decreases supply."
> — `2026_08_26`, ~00:20:50

> "In Solana, adding lamports to an account does not need permission; only withdrawing does."
> — `2026_09_02`, ~00:30:34–00:30:57

> "Reason from the action first, then the state, then the accounts."
> — Recurring instructor heuristic. `2026_08_31`; `2026_09_02`

## Anchor and Rust

> "Box only what you need. The compiler tells you how many bytes you are over stack."
> — `2026_09_02`, ~00:09:28–00:10:44

> "Account types are categories; constraints are rules on top of those categories."
> — `2026_09_02`, ~00:15:32–00:21:17

> "Sometimes you have to look at the data yourself — break it down and deserialize the data yourself."
> — `2026_09_14`, ~00:32:05–00:32:28

> "Anchor already gives us the constraints... we don't really do the manual CPI like we did [for confidential transfers]."
> — `2026_09_18`, ~00:40:14–00:40:26 [ASR corrected: "Angkor" → Anchor]

> "You will not be able to use always the latest and the greatest just because it's available... it is important for you guys to deal with all of these versioning things."
> — `2026_09_21`, ~01:09:52–01:10:07

## Testing

> "Any transaction could randomly pass. Assert expected behavior: balances, lamports, token amounts."
> — `2026_09_04`, ~00:42:33

> "We want to test for both the cases... we want to test for the failure case as well."
> — `2026_09_21`, ~01:11:18–01:11:22

## Token-2022 and extensions

> "It's not waiting for Anchor to hand you the mint... you take the raw bytes and interpret them as a mint containing extensions."
> — `2026_09_14`, ~01:13:45–01:14:10

> "You allocate the space, you create the account, you initialize the extensions before you initialize the mint itself."
> — `2026_09_16`, ~00:17:28–00:17:47

> "Once you've initialized the mint, it will be impossible for you to add any extension."
> — `2026_09_16`, ~00:11:39–00:11:59

> "Always think about the staging area. Anything that has to do with transfer, you have to go through that staging area."
> — `2026_09_16`, ~00:34:08–00:34:26

> "In a confidential token account, what you have is a cryptographic state — not zeros and ones, an encrypted representation of the balance."
> — `2026_09_16`, ~00:24:43–00:25:07

> "The owner is not always the authority."
> — `2026_09_14`, ~00:47:00–00:47:05

> "There are three fundamental ways for a dApp to transfer the funds... the third scenario... you insert an opaque program instruction and the program performs a CPI transfer with the user authorization... you do not know what's contained in the program instruction that you've inserted... it's a black box."
> — `2026_09_18`, ~00:11:51–00:14:12

> "CPI guard is just an extension, token [2022] extension that prohibits setting actions inside CPIs especially when you're dealing with programs that aren't the system program or token programs, so protects users from implicitly signing actions they cannot see."
> — `2026_09_18`, ~00:16:20–00:17:12 [ASR corrected: "CPI card" → CPI guard, "token turn to" → Token-2022]

> "Permanent delegates can transfer tokens with or without the CPI guard... it's kind of bypass the CPI guard."
> — `2026_09_18`, ~00:19:28–00:19:46 [ASR corrected: "CPI card" → CPI guard]

> "The permanent delegate extension doesn't work with token stored in the confidential balance of the token account... if you're working with confidential transfers you don't need to worry about the permanent delegate extension."
> — `2026_09_18`, ~00:20:04–00:20:28

> "The signer is the permanent delegate who is the authority and it can move or burn tokens from every account of its [mint] without consent and without any form of revocation."
> — `2026_09_18`, ~00:45:50–00:46:10 [ASR corrected: "meet" → mint]

> "If you're accepting arbitrary token [2022] mints... you want to create the permanent delegate extension as a reason to reject that mint. Because if you have a mint with a permanent delegate extension, it's more of a problem than a feature."
> — `2026_09_18`, ~00:46:58–00:47:16 [ASR corrected: "means" → mints]

## NFTs and Metaplex Core

> "Core's whole point is it actually kind of cleared out all the problems that other standards have, but still all of them have a place of their own."
> — `2026_09_21`, ~00:08:28–00:08:42

> "You get the benefits you want, but still didn't compromise on the usability."
> — `2026_09_21`, ~00:12:38–00:12:44

> "Staking is where you have something which you lend to someone else, you lock it, and you get something in return... some kind of reward for actually locking your asset."
> — `2026_09_21`, ~00:31:21–00:31:40 [lightly paraphrased for readability]

> "We are not storing any data in there, there is no point in actually creating that account in the first place... the only feature we want out of that account is it being able to sign."
> — `2026_09_21`, ~00:39:20–00:39:54, on using a PDA as update authority

> "Just because I say this is an admin doesn't mean anything actually... in production level applications you probably want to actually separate a maintained whitelist of accounts."
> — `2026_09_21`, ~00:43:48–00:44:02

> "There is no other way around it: either you've got to do it yourself, or you've got to be with someone who does it for you and incentivize them in some way."
> — `2026_09_21`, ~01:29:53–01:30:10, on sourcing niche oracle data

## DeFi / markets

> "Arbitrage is just buying low and selling high across inefficient markets."
> — `2026_09_07`, ~00:32:59–00:33:05

> "Front-running and sandwiching are a whack-a-mole problem; every defense invites a more sophisticated attack."
> — Paraphrased from `2026_09_07`, ~00:51:55–00:52:28

## RWA / tokenized fund

> "How the NAV value is set — that is all off-chain. From the protocol, all I care about is NAV value."
> — `2026_09_25`, ~00:35:28–00:36:10, on the on-chain/off-chain boundary

> "That's the end of the cohort. It's not the end of building. You're already five weeks in... usually people who make it are those who are consistent."
> — `2026_09_25`, ~01:36:21–01:36:47

> "Usually people who make it are those who are consistent. You figure out all your learnings, you take it and try to build something new or try to improve on something."
> — `2026_09_25`, ~01:36:35–01:36:47

## LOI, capstone, requirements

> "Business planning is never set in stone. An LOI is a draft, not a cemented commitment."
> — `2026_09_07`, ~00:04:20–00:05:14

> "Don't use AI for the initial thinking. Build the framework yourself, then red-team it with AI."
> — `2026_09_07`, ~00:12:39–00:13:50

> "If you don't know what you're doing, share your screen first. That's when the seniors can actually help you learn."
> — `2026_09_09`, ~01:19:02–01:21:00

> "Don't clone the reference repo. Initialize from scratch, use the reference as a guide, and document what every constraint does."
> — Paraphrased from `2026_09_09`, ~01:21:36–01:22:46

> "There's a way to get through it... sit down with your new teammates, hash out a plan, divide and conquer, have calls where you are syncing every day."
> — `2026_09_23`, ~00:09:08–00:09:30, on capstone execution

> "Don't fix it on any specific implementation just because [the student] does it a specific way... Do whatever you want."
> — `2026_09_23`, ~01:37:31–01:37:38, on the claim-rewards design

> "This is the key to the castle — this is what is going to allow you to write your code in a manner of two or three days and make it testable and objective."
> — `2026_09_28`, ~00:08:17–00:08:36, on atomic requirements

> "We shouldn't be drawing until we have requirements."
> — `2026_09_28`, ~01:08:11–01:08:16

> "Every requirement needs to be atomic, just like every blade of grass you touch."
> — `2026_09_28`, ~01:20:13–01:20:26

> "If this is hard for you, how the hell are you coding?"
> — `2026_09_28`, ~01:15:19–01:15:37, on requirement granularity being the real work

> "Anywhere you are is fine, but not starting isn't."
> — `2026_09_28`, ~00:26:29–00:26:36

## Ecosystem

> "Newer standards may be technically superior, but check whether explorers and apps actually support them."
> — Paraphrased from `2026_08_28`, ~01:04:43–01:05:24
