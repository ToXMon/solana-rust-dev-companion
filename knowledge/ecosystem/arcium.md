# Arcium: Confidential Computing Network for Solana

> Multi-Party Computation (MPC) infrastructure enabling encrypted computation on Solana without exposing data to any single node.

Arcium is a production-grade encrypted supercomputer live on Solana mainnet (v0.15.0 as of September 2026). It allows Solana programs to execute operations on encrypted data via off-chain MPC clusters while maintaining privacy through secret sharing.

---

## Core Concepts

**What Arcium is:**
Arcium enables computation on encrypted data. Client sends encrypted inputs to a Solana program, which queues a computation to an MPC cluster (Arx nodes), receives encrypted results, and triggers a callback instruction. No data is ever decrypted on-chain; the MXE (MPC eXecution Environment) holds the keys.

**Architecture components:**

1. **Arx nodes:** Decentralized processors that execute MPC circuits; stake to participate; honest-majority model (privacy maintained if ≥1 node honest)
2. **arxOS:** Distributed coordination layer managing off-chain execution
3. **Arcis:** Rust-based framework for writing MPC circuits (deterministic, fixed-size, no dynamic behavior based on encrypted data)
4. **MXEs:** Solana accounts binding computation definitions to a cluster and program

**How encryption works:**
- Client performs X25519 key exchange with MXE, then encrypts inputs with Rescue cipher → ciphertext
- Ciphertext is converted to secret shares distributed across Arx nodes
- Each node holds random-looking fragments; no individual node learns plaintext
- Arx nodes compute collaboratively on shares (MPC protocol)
- Results encrypted and returned; client decrypts with shared secret

---

## Solana Program Integration

Arcium programs are standard Anchor programs using the `#[arcium_program]` macro. Data flow:

```
Anchor program (public) → CPI to Arcium program
  ↓
Queue computation (encrypted inputs + callback instruction)
  ↓
Arcium program stores on-chain
  ↓
Arx cluster (off-chain) executes circuit on encrypted data
  ↓
MPC returns encrypted result → triggers callback instruction
  ↓
Anchor program consumes encrypted output (via event)
```

**Key fact:** Encrypted instructions (circuits) live in `encrypted-ixs/` directory (Arcis/Rust). The Solana program invokes them via CPI; results come through callback.

---

## Arcis Primitives

All primitives run as optimized MPC circuits (not native Rust).

### Random Number Generation

```rust
// Boolean (50/50 probability)
let coin_flip = ArcisRNG::bool();

// Integer with specific bit width
let random_byte = ArcisRNG::gen_integer_from_width(8);  // 0-255
let random_u64 = ArcisRNG::gen_integer_from_width(64);

// Uniformly random value
let random_array = ArcisRNG::gen_uniform::<[u8; 32]>();

// Range-based (rejection sampling)
let (roll, success) = ArcisRNG::gen_integer_in_range(1, 6, 24);
// success = true with >50% per attempt, failure probability < 2^-24 with 24 attempts

// Shuffle arrays (O(n·log³(n) + n·log²(n)·sizeof(T)))
ArcisRNG::shuffle(&mut cards);
```

Secret vs public randomness:
- `gen_integer_from_width()`: secret across nodes
- `gen_public_integer_from_width()`: visible to Arx nodes

### Cryptographic Operations

**SHA3 hashing:**
```rust
let hasher = SHA3_256::new();
let digest = hasher.digest(&message).reveal();  // Returns [u8; 32]

let hasher = SHA3_512::new();
let digest = hasher.digest(&message).reveal();  // Returns [u8; 64]
```

*Note: Arcis uses SHA3 (Keccak) instead of SHA-2 for more efficient MPC circuits.*

**Ed25519 signatures:**
```rust
// Verify signature
let vk = verifying_key.unpack();
let sig = ArcisEd25519Signature::from_bytes(signature);
vk.verify(&message, &sig).reveal()  // Returns bool

// Generate keypair (secret key stays secret-shared)
let secret_key = SecretKey::new_rand();
let verifying_key = VerifyingKey::from_secret_key(&secret_key);
verifying_key.reveal()  // Only public key revealed

// Sign with MXE cluster's collective key
MXESigningKey::sign(&message).reveal()  // Returns signature
```

**X25519 public key operations:**
```rust
let pk1 = ArcisX25519Pubkey::from_uint8(&key1);
let pk2 = ArcisX25519Pubkey::from_uint8(&key2);
(pk1 == pk2).reveal()  // Returns bool

// From base58
ArcisX25519Pubkey::from_base58(b"2uKu51kQaLseu7FySMAGWU6hpnjNvgGr3PkvUCBVTTPD")

// Extract/rebuild from Montgomery X-coordinate (for ECDH/KDF)
let x = pubkey.to_x();
let rebuilt = ArcisX25519Pubkey::new_from_x(x);
```

### Field Arithmetic (BaseField25519)

Modulo 2^255 - 19 (native field element for cryptographic operations):

```rust
// Construction
let a = BaseField25519::from_u128(42);
let b = BaseField25519::from_i64(-1);
let p = BaseField25519::power_of_two(255);

// Arithmetic (wraps modulo 2^255-19)
let sum = a + b;
let product = a * b;
let difference = a - b;
let negative = -a;

// Comparisons (unsigned: [0, p-1])
a < b  // Returns bool

// Division
let inverse = a.safe_inverse();  // Returns 0 for inverse of 0
let quotient = a.field_division(b);  // Division by 0 returns 0
let euclidean = a.euclidean_division(b);  // Panics on division by 0

// Serialization
let bytes = a.to_le_bytes();  // [u8; 32] little-endian
```

### Machine Learning Primitives

```rust
// Binary classification
let model = LogisticRegression::new(weights, bias, features);
let prediction: bool = model.predict();
let probability: u128 = model.predict_proba();

// Regression
let model = LinearRegression::new(weights, bias, features);
let prediction = model.predict();

// Math
let result = ArcisMath::sigmoid(x);
```

### Data Packing

Efficient fixed-size storage for cryptographic data:
```rust
let packed = Pack::<VerifyingKey>::new(key);
let unpacked = packed.unpack();
```

---

## Verifiable Randomness: Arcium vs Switchboard

Arcium provides randomness; Switchboard provides **verifiable** randomness (the critical difference for lotteries).

**Arcium RNG:**
- Generated deterministically within MPC cluster
- Encrypted and returned to Solana
- NOT cryptographically verifiable on-chain
- Requires trusting Arx nodes executed circuit honestly
- Suitable for: internal computations where privacy > external proof

**Switchboard VRF:**
- Off-chain oracle network generates random value
- Oracles sign with BLS signature
- Signature posted on-chain; Solana program verifies it
- Cryptographically verifiable by anyone
- Suitable for: lotteries, draws, anything requiring fairness proof

| Factor | Arcium RNG | Switchboard VRF |
|--------|-----------|-----------------|
| On-chain verification | ✗ NO | ✓ YES (BLS signature) |
| User can independently verify | ✗ NO | ✓ YES |
| Latency | 5-30 min (MPC consensus) | 10-60 sec (oracle network) |
| Cost | Wrapped in computation | ~0.05 SOL per draw |
| Trust model | MPC dishonest majority | Oracle network + cryptography |

**For lottery/draw use case (Cryptoball):** Use Switchboard VRF for randomness generation; use Arcium for privacy-critical follow-up logic if needed.

---

## Security Model

**Dishonest majority (MPC foundational):**
Privacy maintained as long as ≥1 Arx node remains honest, even if all others collude. If you run your own node in a cluster, you guarantee honest participation.

**v0.15.0 improvements:**
- Distributed preprocessing (Cerberus replaces single trusted dealer)
- MXE authority transfer (explicit key recovery for multisig vaults)
- Per-peer stake snapshotting (stable fee splits across membership changes)

**Fault handling:**
Cerberus protocol detects faults and aborts computation rather than corrupting results. Applications must handle abort and retry.

**Known constraints to audit:**
- Memory bounds in fixed-size buffers (no Vec; use arrays)
- Bit-width overflow (field elements vs bounded integers)
- Information leakage via execution time (comparisons are expensive)
- Deserialization of untrusted outputs (verify signatures/commitments)
- MPC protocol faults (application must handle abort)

---

## Development Setup

**Prerequisites:** Rust 1.70+, Solana CLI, Anchor, Node.js/npm

**Install Arcium toolchain:**
```bash
curl https://arcium.sh | sh
arcup install latest
arcup use latest
arcium --version  # Should show v0.15.0 or later
```

**Initialize project:**
```bash
arcium init my-mxe-project
cd my-mxe-project
```

Project structure:
```
├── Arcium.toml             # Config (cluster offset, versioning)
├── encrypted-ixs/          # Arcis circuits (Rust/MPC)
│   └── add_together.rs    # Example circuit
├── programs/my_mxe/        # Solana program (Anchor + #[arcium_program])
├── tests/hello_world.ts    # TypeScript integration tests
└── package.json
```

**Build & test:**
```bash
arcium build
arcium test                    # Local cluster
arcium test --cluster devnet   # Devnet
arcium test --cluster mainnet  # Mainnet (testnet)
```

---

## Mental Model: MPC Constraints

Understanding these constraints is essential for writing Arcis circuits.

**Circuit structure is fixed at compile time:**
Circuits compile to fixed structure; secret data flows through at runtime. Consequence: **no dynamic behavior based on encrypted data.**

**Fixed-size data only:**
No `Vec<T>`, `String`, or `HashMap`. Use `[T; N]` arrays; memory footprint must be known at compile time.

**Both branches execute (when condition is secret-dependent):**
```rust
if secret_condition {
    expensive_op()  // Always runs
} else {
    cheap_op()      // Always runs
}
// Cost = cost(expensive) + cost(cheap), NOT max
```

**Fixed loop bounds:**
```rust
for i in 0..100 { }        // ✓ Works
while secret < 100 { }     // ✗ Won't compile
```

**Reveal placement:**
`.reveal()` and `.from_arcis()` cannot be inside secret-dependent conditionals; they are global side effects.

**No early exit:**
No `break`, `continue`, or `return` inside loops; restructure into functions with single exit.

**Cost ranking (MPC vs standard Rust):**
| Operation | Cost |
|-----------|------|
| Addition, subtraction | Nearly free |
| Multiplication by constant | Nearly free |
| Multiplication | Cheap |
| Comparisons (`<`, `>`, `==`) | Expensive |
| Division, modulo (general) | Very expensive |
| Dynamic array indexing | O(n) |
| Sorting | O(n·log²(n)·bit_size) |

---

## Known Limitations (v0.15.0)

1. **Output size:** Encrypted outputs capped at ~5KB (single PDU)
2. **Latency:** 5-30 minutes from queue to callback (MPC consensus + on-chain finalization)
3. **Devnet Arx nodes:** Limited availability; mainnet more stable
4. **Unsupported Rust features:** `let ... else`, `Vec`, `String`, `HashMap`, floats
5. **Limited ML:** Basic logistic/linear regression only in circuits
6. **Circuit size:** Complex circuits may timeout

**Workarounds:**
- Batch frequent operations into single computation
- For real-time: hybrid (Arcium privacy-critical, instant decisions off-chain)
- For large output: split into multiple computations
- For floats: fixed-point arithmetic (u128 with manual scaling)

---

## Mainnet & Devnet Status

Arcium **live on Solana mainnet** as of June 2026 (v0.11.0 baseline).

| Component | Mainnet | Devnet | Status |
|-----------|---------|--------|--------|
| Arcium program | ✓ | ✓ | Production |
| Arx nodes | ~15-20 active | Several | Operational |
| Circuit compiler (arcup) | ✓ | ✓ | v0.15.0 |
| TypeScript SDK | ✓ | ✓ | Production (`@arcium-hq/*`) |
| Examples | ✓ | ✓ | Runnable |

**Deployment:** Upgrade to v0.15.0 (or v0.14.1 minimum). Existing MXEs remain compatible.

---

## Resources

**Official docs:** https://docs.arcium.com

**Key pages:**
- [Installation](https://docs.arcium.com/developers/installation.md)
- [Hello World](https://docs.arcium.com/developers/hello-world.md)
- [Core Concepts](https://docs.arcium.com/developers/core-concepts.md)
- [Arcis Mental Model](https://docs.arcium.com/developers/arcis/mental-model.md)
- [Primitives API](https://docs.arcium.com/developers/arcis/primitives.md)
- [Operations Reference](https://docs.arcium.com/developers/arcis/operations.md)
- [Best Practices](https://docs.arcium.com/developers/arcis/best-practices.md)
- [Deployment Guide](https://docs.arcium.com/developers/deployment.md)

**GitHub:** https://github.com/arcium/

**TypeScript SDK:** `@arcium-hq/client`, `@arcium-hq/reader`, `@arcium-hq/staking`

**Crates:** Arcium Rust crates on crates.io (see release notes for versions)

---

## When to Use Arcium

✓ **Good fit:**
- Confidential state machines (private portfolio tracking, sealed auctions)
- Privacy-preserving computation (fairness without exposure)
- Encrypted access control (decryption only inside MPC)
- Private data analysis (aggregations without revealing individual data)

✗ **Poor fit:**
- Real-time operations (5-30 min latency unsuitable)
- Lotteries (use Switchboard VRF for verifiable randomness)
- Simple public operations (overhead not justified)
- High-frequency operations (batch into fewer computations instead)
