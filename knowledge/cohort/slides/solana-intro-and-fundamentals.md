# Slide deck extract: Turbin3 Builders Q3 26 — Solana intro and fundamentals

Source: Turbin3 Builders Q3 26- Solana intro and fundamentals.pdf
Method: pypdf text extraction; diagrams/images are not transcribed.

=== PAGE 1 ===
TURBIN3.ORG
Solana
Fundamentals
Q3 2026


=== PAGE 2 ===
TURBIN3.ORG
Overview
Solana 
Accounts
Programs
Instructions
Transactions
PDAs
ATAs
CPI


=== PAGE 3 ===
TURBIN3.ORG
What is Solana?
● Global network of computers working together without a single point of 
failure. These computer are called validators 
● Process transactions in less than a second
● Run decentralized applications with 100% uptime at a very low cost 
(typically at around $0.00025- you could make 4k txs for less than $1)
● Maintain a verifiable shared records of all activity
● Globally accessible and open to anyone


=== PAGE 4 ===
TURBIN3.ORG
What is Solana?
How it works:
Action -> Transaction -> Consensus -> Confirmation -> Result (state change)


=== PAGE 5 ===
TURBIN3.ORG
Accounts
● Fundamental unit of data on Solana
● Everything on Solana is an account
● Key-value pair mappings
● Only the owner of an account can modify its data or debit any lamports. 
Any program can credit lamports to any writable account.
● Every account is required to hold a minimum lamport balance 
proportional to its data size to remain onchain. 


=== PAGE 6 ===
TURBIN3.ORG
Accounts


=== PAGE 7 ===
TURBIN3.ORG
Accounts
● lamports  (1 SOL = 1,000,000,000 lamports)
● data (Account state or program bytecode)
● owner 
● executable (True= program account, False= state account)
● rent_epoch (now we set to u64 ::MAX for rent exempt accounts)
● Each account is identified by a unique 32-byte address 
(base58-encoded string)
Question for thought: Can the owner be reassigned or is it immutable?


=== PAGE 8 ===
TURBIN3.ORG
Accounts
● Program account: stores executable code. Every program account is 
owned by a loader program
● Program data accounts
● Data Accounts (program state account): These store only 
program-defined state or user data. This involves two steps: 
1. Invoke System program to create the account. The System Program 
transfers ownership to the specified program. 
2, The owning program initializes the account’s data field according to its 
instructions


=== PAGE 9 ===
TURBIN3.ORG
Accounts
● System accounts: remained owned by System programs after creation. 
Sending Lamports to a new address for the first time creates a new 
account at that address (owned by?)
● Fee payer on a transaction must be a system account. Only system 
program owned accounts can pay tx fees
● Sysvar Accounts: special accounts that provide read-only access to 
cluster state. They are dynamically updated per slot


=== PAGE 10 ===
TURBIN3.ORG
Think about? 
● what actually determines whether an account is rent-exempt? Is rent_epoch 
= u64::MAX a cause of exemption or a signal that gets set as a consequence 
of something else being true?
● For a program deployed under the upgradeable loader 
(BPFLoaderUpgradeable) which is now the default path, what does the 
program account itself actually contain, and what does it point to? Does the 
executable=true account hold the bytecode, or something else?
● For programs deployed under the non-upgradeable BPFLoader?

=== PAGE 11 ===
TURBIN3.ORG
Programs
● A Solana program is an account that contains executable sBPF 
bytecode and has exectuable = True
● Stateless 
● Upgradable: programs deployed with loader -v3 (BPF Loader 
Upgradeable) can be upgraded when an upgrade authority is set. It 
becomes immutable when that authority is revoked.
● Anchor (framework uses Rust macros to reduce boilerplate). Native Rust 
(gives full control but more manual and granular implementation)


=== PAGE 12 ===
TURBIN3.ORG
Programs
● Native Loader: owns builtins (System Vote, Stake and other loaders)
● BPF Loader (V1): Legacy programs
● BPF Loader (V2): Legacy programs
● BPF Loader upgradeable: Owns all newly deployed programs. 
● Syscalls: Programs interact with the runtime through these. They cover 
things such as logging, hashing, CPI, cryptography, mem ops, etc.


=== PAGE 13 ===
TURBIN3.ORG
Core Programs
● These provide fundamental functionality. For example, account 
management, consensus, transaction optimization, privacy.
● System program: creates new accounts. All new accounts are owned by 
the system program with ownership typically re-assigned upon creation. 
Transfer SOL. Allocate data.
● Vote program: Manage accounts that track validator voting 
state/reward
● Stake program: creates and manages stake delegations
● Config: Stores configuration data onchain
● Compute budget: Set compute unit limits and priority fees for Txs
● ZK ElGamal Proof: Verifies zk proofs for ElGamal-encrypted data
● Address-Lookup Table: manages ALTs for txs that reference many 
accounts 


=== PAGE 14 ===
TURBIN3.ORG
Instructions
● Instruction is a request to execute a specific function on a Solana 
program. They are the basic component for onchain ops.
● specifies : one program to call, the accounts it needs, and a byte array of 
data that the program decodes. 
● Logic for each instruction is stored on a program, each program defines 
its own set of instructions
● Transactions which are sent to the network to be processed are made 
up of one or more instructions


=== PAGE 15 ===
TURBIN3.ORG
Instructions
Pub struct instruction {
/ / /pubkey address of the program that executes this instruction
pub program_id: Pubkey,
/ / / describes accounts that should be passed to the given program
pub accounts: Vec <AccountMeta>,
/ / / byte array with additional data to be used by instruction
pub data: Vec <u8>, 
}


=== PAGE 16 ===
TURBIN3.ORG
Instructions
Pub struct AccountMeta {
/ / /An accounts public key
pub pubkey: Pubkey,
/ / / true if an instruction require a Tx signature matching pubkey 
pub is_signer : bool,
/ / / true if the instruction modifies account data during execution
pub is_writable: bool, 
}


=== PAGE 17 ===
TURBIN3.ORG
Instructions
● To know which accounts an instruction requires, including which must be 
writable, read-only or sign the tx, refer to the implementation of the 
instruction as defined by the program
● In practice, as we will see, you don’t have to construct an Instruction 
manually. Most programs provide client libraries with helper functions 
that create instructions for you. But if the library is available, you can 
manually build the instruction yourself.


=== PAGE 18 ===
TURBIN3.ORG
Transactions
● A transaction comprises of one or more instructions, the signature of 
accounts that authorizes the changes, and a recent blockhash.
● The network processes all instructions in a transaction together. (Why?)
● The size of a transaction is 1,232 bytes but raised to 4,096 in v1 
versioned transactions
● Each signer in a transaction provides one 64-byte Ed25519 signature
● The recent lockhash is valid for a short period (usually 150 slots). 
● A transaction only executes if it carries a valid signature for every 
required signer. 
Question for thought: Where does private key live and who is allowed 
to authorize a translation with it?


=== PAGE 19 ===
TURBIN3.ORG
Transactions
Pub struct Transaction {
/ / / An array of signatures
pub signatures: Vec<Signatures>,,
/ / /  msg containing tx information, including list of instructions
pub message :  Message,
}


=== PAGE 20 ===
TURBIN3.ORG
Transactions
● One signature is required for every signer account referenced by the 
transaction instruction.
● The first signature in the array belongs to the fee payer (fee payer?)
● The first signature is also used as the transaction ID used to look the 
transaction on the network. It is most commonly referred to as the 
transaction signature.
● Fee payer must be first account in the message and a signer.


=== PAGE 21 ===
TURBIN3.ORG
Transactions
Pub struct Message {
/ / / the messageheader to identify signed/read-only account keys
pub header: MessageHeader,
/ / /  All account keys used by this transaction
pub account_keys : Vec<Pubkey>,
/ / / The id of a recent entry on the ledger
pub recent_blockhash: Hash,
/ / / programs that will be in sequence and committed in one atomic tx
pub instructions: Vec<CompiledInstructions>
}


=== PAGE 22 ===
TURBIN3.ORG
pub struct MessageHeader {
    / / / The number of signatures required for this message to be considered    
valid. The signers of those signatures must match the first 
num_required_signatures` of [`Message::account_keys`].
    pub num_required_signatures: u8,
    / / / The last `num_readonly_signed_accounts` of the signed keys are 
read-only accounts.
    pub num_readonly_signed_accounts: u8,
    / / / The last `num_readonly_unsigned_accounts` of the unsigned keys are
    read-only accounts.
    pub num_readonly_unsigned_accounts: u8,
}

=== PAGE 23 ===
TURBIN3.ORG


=== PAGE 24 ===
TURBIN3.ORG
Recent Blockhash
● This field is a 32-byte hash that serves two purposes:
1. Timestamp the txs to prove it was created recently
2. Prevent the same txs from being processed twice
● Expires after 150 slots. 


=== PAGE 25 ===
TURBIN3.ORG


=== PAGE 26 ===
TURBIN3.ORG
PDAs 
● Program derived addresses (PDAs) are 32-byte account address that are 
deterministically derived from a program ID and a set of seeds
● They are guaranteed to not lie on the Ed25519 curve, and therefore have no 
private keys. 
● Only the program whose ID was used in the derivation can sign for a PDA, 
through a syscall in a process called cross program invocations.
● PDAs are used when you need to derive the same accounts from the same 
seeds every single time, when you want programs to act as autonomous 
authorities, for user-scoped state (derive per-user accounts from user 
pubkey seeds, and most importantly, when no keypair management is 
needed.

=== PAGE 27 ===
TURBIN3.ORG
PDAs 
● PDAs are derived by hashing seeds and program ID and a bump via SHA-256 
until the result is off the Ed25519 curve
● The canonical bump is the first value that produces an off-curve address
● For seeds:
Maximum 16 seeds per derivation
Maximum 32 bytes per seed 
● Bump seed is a single byte (u8) appended to the optional seeds during 
derivation. find_program_address searches from 255 down to 1, calling 
create_program_address with each value until the result falls off the Ed25519 
curve. The first value that succeeds is the canonical bump.
● Programs should always use the canonical bump to ensure a unique, 
deterministic mapping from seeds to address.

=== PAGE 28 ===
TURBIN3.ORG
PDAs 
● Deriving a PDA and creating an account at a PDA are two separate 
operations. You must explicitly create the account after deriving the address
● To create an account at a PDA, the deriving program invokes the System 
Program create_account instruction via CPI, passes the PDA’s seeds so the 
runtime can verify the program authority over that address

=== PAGE 29 ===
TURBIN3.ORG
CPIs 
● Cross program Invocation happens when one program calls an instruction on 
another program during execution
● This is useful (why?)
● When a CPI doesn’t require PDA signers, the invoke function is used
● When a CPI requires a PDA signer, the invoke_signed function is used with 
the signer seeds used to derive the PDA


=== PAGE 30 ===
TURBIN3.ORG
Think about? 
● What are important fee payer validation checks?
● What are security implications that may arise if programs do not use the 
canonical bump when deriving PDAs
● Consider this case: a program creates a PDA via invoke_signed calling 
System Program's create_account. Does that PDA pass through a state 
where System Program owns it, and then get reassigned to the calling 
program afterward, or does create_account let you set the final owner 
directly, in one atomic step? Why would the two-step version (create, then 
assign) even be risky in a program context, security-wise?

=== PAGE 31 ===
TURBIN3.ORG
Questions?


=== PAGE 32 ===
TURBIN3.ORG
Follow us for updates, insights, educational content, and community 
engagement:
X (Twitter): https://x.com/solanaturbine
LinkedIn: https://linkedin.com/company/turbin3
YouTube: https://youtube.com/@Turbin3
