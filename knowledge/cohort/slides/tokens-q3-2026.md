# Slide deck extract: Tokens Q3 2026 (Turbin3 Builders Cohort)

Source: tokens Q3 2026.pdf
Method: pypdf text extraction; diagrams/images are not transcribed.

=== PAGE 1 ===
Aug 26, 2026
Solana Tokens Operate 
Through Specialized 
Programs and Accounts
An introduction to token architecture.
Agenda
01 The Token Program
02 Mint Accounts
03 Token Accounts
04 Associated Token 
Accounts (ATA)
05 Token-2022 Extensions
TURBIN3.ORG

=== PAGE 2 ===
TURBIN3.ORG
The Token Program Executes All
Token Logic and State Changes
The Solana Token Program is a native program responsible for managing 
the state of all tokens on the network. It provides standard instructions to 
create new tokens, mint supply, transfer balances, and burn tokens.
Crucially, the Token Program itself does not hold any user balances. 
Instead, it processes transactions and updates the state stored in 
separate, program-owned data accounts.

=== PAGE 3 ===
TURBIN3.ORG
Mint Accounts Define the Global State of a Specific 
Token
A Mint Account acts as the blueprint for a specific asset on Solana. It stores the global 
configuration for the token, ensuring one Mint address equals one unique token type.
Key data stored within a Mint Account includes:
● Total circulating supply
● Number of decimals used for fractional amounts
● Mint authority allowed to create new tokens
● Freeze authority capable of locking accounts

=== PAGE 4 ===
TURBIN3.ORG
Token Accounts Track Individual User Balances and 
Ownership
While the Mint Account defines the token, Token Accounts are required to actually hold and transfer 
balances.
01 / OWNER WALLET
Establishes a relationship with 
a specific user wallet as the 
authorized owner.
02 / TOKEN MINT
Binds the account directly to 
a specific parent token mint 
on the network.
03 / CURRENT BALANCE
Tracks and maintains the 
exact active token balance 
independently.
Scale & Multiplicity: A single Mint Account can map to millions of individual Token Accounts across the 
network, each independently tracking the holdings of different users.

=== PAGE 5 ===
TURBIN3.ORG
Associated Token Accounts
Standardizing Token Storage for Wallets
The Associated Token Account (ATA) program 
resolves the complexity of users holding multiple 
token accounts for the same asset.
By using a deterministic derivation, it computes a 
canonical address using only:
• User's Main Wallet Address
• Token's Mint Address
This guarantees exactly one primary Token 
Account per token, streamlining transfers and 
preventing fragmented balances.
DETERMINISTIC DERIVATION (PDA)
User Wallet
PublicKey Address
Token Mint
PublicKey Address
PDA
CANONICAL
ATA
ADDRESS
Guarantees exactly one primary Token Account 
per asset, eliminating balance fragmentation.

=== PAGE 6 ===
TURBIN3.ORG
Executing a Transfer Requires Updating Token 
Accounts
01 // INITIATE
Transfer Request
When Alice transfers 10 
USDC to Bob, her 
wallet sends a transfer 
instruction to the Token 
Program.
02 // VALIDATE
Token Program
The Token Program 
validates Alice's 
signature and ensures 
she has sufficient 
funds.
03 // UPDATE STATE
Balance Updates
Upon validation, the 
program decreases the 
balance in Alice's USDC 
ATA by 10 and 
increases Bob's USDC 
ATA by 10.
04 // NO CHANGE
Mint Integrity
The Mint Account 
remains unchanged 
throughout this 
process.

=== PAGE 7 ===
TURBIN3.ORG
Minting, Burning & Supply
Three fundamental state transitions dictate token supply and distribution.
01 // MINT
Increase Supply
Increases account balance 
and total supply.
02 // TRANSFER
Shifts Balances
Shifts balances; supply 
remains constant.
03 // BURN
Decrease Supply
Decreases account balance 
and total supply.

=== PAGE 8 ===
TURBIN3.ORG
Token-2022 Introduces Advanced Extensions to the 
Standard Model
// OVERVIEW
The Token-2022 program is an upgraded 
version of the original Token Program 
designed to support advanced tokenomics 
without breaking composability.
It introduces customizable extensions that 
can be added directly to both Mint and 
Token Accounts.
// KEY_EXTENSIONS
> Transfer Fees: Royalty collection mechanisms
> Transfer Hooks: Custom logic during 
movement
> Native Metadata: Built-in token metadata 
standard
> Default State: Account state configurations
> Permanent Delegates: Controlled compliance 
controls

=== PAGE 9 ===
TURBIN3.ORG
Token-2022 Operates Parallel to the Original Token 
Program
ORIGINAL TOKEN PROGRAM
Coexists on Solana utilizing its own 
traditional program ID.
Provides the standard baseline for 
blockchain transactions: standard minting 
mechanics, traditional token accounts, 
and basic asset transfers.
TOKEN-2022 PROGRAM
Allows developers to enable highly 
modular extensions built directly at the 
Mint level.
Requires wallets and decentralized 
applications to explicitly support the new 
program ID to interact with these 
extended assets.