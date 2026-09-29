# Slide deck extract: NFTs Q3 2026 (Turbin3 Builders Cohort)

Source: NFTs Q326.pdf
Method: pypdf text extraction; diagrams/images are not transcribed.

=== PAGE 1 ===
Q3 2026
TURBIN3.ORG
The Evolution of
Solana NFTs
> From Token Metadata to Metaplex Core

=== PAGE 2 ===
TURBIN3.ORG
What is an NFT?
NFTs bring ownership, identity, and programmability to digital assets on Solana.
● Uniquely Identifiable: Distinct digital assets with verifiable on-chain 
ownership.
● Architecture: On-chain ledger tracks ownership; metadata accounts define 
properties.
● Versatile Applications: Digital art, gaming assets, tickets, and credentials.

=== PAGE 3 ===
TURBIN3.ORG
The Original Standard
The legacy Token Metadata standard adapted a fungible token system to represent 
unique assets, introducing architectural complexity.
Mint Account ➔
 Token Account ➔
 Metadata Account
● Mechanics: SPL Token manages transfer; Metaplex 
layer adds identity and collections.
● Limitation: Requires managing multiple accounts; 
not natively designed for single unique assets.
Key Takeaway
Early standards adapted 
fungible token architecture 
for non-fungible assets.

=== PAGE 4 ===
TURBIN3.ORG
Programmable & Compressed NFTs
To address constraints in enforcement and costs, the ecosystem introduced 
Programmable (pNFT) and Compressed (cNFT) standards.
> pNFTs
Enforce transfer rules and locking 
behaviors on top of Token Metadata.
> cNFTs
Use on-chain Merkle trees to compress 
state, drastically cutting storage costs.
// Use Cases
Optimized for large-scale gaming and mass 
distribution.
// Key Takeaway
pNFTs improved rule enforcement; cNFTs 
unlocked massive scale.

=== PAGE 5 ===
TURBIN3.ORG
Solana NFT Evolution
Each evolution directly solved architectural limitations of the standard before it.
01
Token Metadata
Adapted fungible 
tokens.
Early standard built on top 
of the original SPL model, 
linking external metadata 
URIs to support 
collectibles.
02
Programmable 
NFTs
Enforced on-chain 
rules.
Addressed royalty evasion 
by embedding 
customizable rulesets 
directly into transfer and 
minting logic.
03
Compressed NFTs
Merkle tree 
compression.
Slashed minting overhead 
to micro-costs using state 
compression tech and 
off-chain ledger trees.
04
Metaplex Core
Unified, extensible 
primitive.
Next-gen model designed 
specifically for NFTs, 
completely bypassing 
legacy SPL account 
complexity.

=== PAGE 6 ===
TURBIN3.ORG
Metaplex Core
Metaplex Core is a purpose-built standard that simplifies digital assets by using a single unified 
account. Core treats the NFT as the primary primitive, abandoning the fungible-token shell.
Legacy Model (SPL Token)
Mint
 Token
 Metadata
Replaced complex structure requiring 3 separate 
accounts.
Metaplex Core Redesign
One Asset = One Account
Drastically reduces rent fee structures and footprint 
on-chain.
Extensibility: Flexible Plugins configure custom on-chain behaviors
Royalties
Enforces asset creator 
share rules directly 
on-chain.
Freezing
Locks transaction and 
movement natively.
Delegates
Manages transfer and 
custom authority 
permissions.
Developer Logic
Implements customized 
external hooks and 
execution triggers.

=== PAGE 7 ===
TURBIN3.ORG
Why Core?
Metaplex Core significantly reduces cost, simplifies developer integration, and delivers a superior 
architectural model.
FEATURE TOKEN METADATA METAPLEX CORE
Architecture Multiple accounts Single Asset account
Cost Higher fees Substantially lower rent costs
Extensibility Complex program logic Modular Plugins
Dev Experience Complex integration Simple, clean path
CORE ADVANTAGES: Simpler ➔ Cheaper ➔ Extensible ➔ Purpose-built