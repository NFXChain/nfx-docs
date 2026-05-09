# System Architecture

## High-Level Overview

NFX Chain employs a **modular microservices-inspired architecture** where each component runs as a separate process but can be linked into a monolithic binary for performance. The system is divided into four main modules:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   Wallet    â”‚â—„â”€â”€â–ºâ”‚    Go API   â”‚â—„â”€â”€â–ºâ”‚  Bindings   â”‚â—„â”€â”€â–ºâ”‚    Core     â”‚
â”‚  (Qt/QML)   â”‚    â”‚  (REST/CLI) â”‚    â”‚   (C ABI)   â”‚    â”‚ (C++/Qt)    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                                                              â”‚
                                                              â–¼
                                                    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                                                    â”‚  libnfx-core.so     â”‚
                                                    â”‚  (Consensus, P2P,   â”‚
                                                    â”‚   Crypto, VM, DB)  â”‚
                                                    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

## Core Module (nfx-core)

### Components

#### 1. Consensus Layer
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚              Consensus Engine                   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  â€¢ PoW Miner          â”‚  PoES Validator         â”‚
â”‚  â€¢ PoS Block Signer   â”‚  AI Governance Engine   â”‚
â”‚  â€¢ Difficulty Adjusterâ”‚  Reward Calculator       â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Proof of Exponential Security (PoES) Algorithm:**

```
Security(S) = 1 - e^(-kÂ·NÂ·t)

Where:
  S = Network security probability
  N = Number of honest nodes
  t = Time in blocks
  k = Security constant (protocol parameter)

An attacker needs 51% of stake + 51% of compute + 
51% of network topology to compromise chain.
```

#### 2. P2P Network
- **Hypernode Mesh**: Each node connects to 8-12 geographically distributed hypernodes
- **Message Types**: `TX`, `BLOCK`, `INV`, `GET_DATA`, `GET_BLOCKS`, `ADDR`, `ALERT`
- **Protocol**: Bitcoin-style with extensions for PoESVerify
- **Encryption**: TLS 1.3 with forward secrecy (ECDHE)

#### 3. Blockchain & Storage
```cpp
// Block structure
struct Block {
    uint32_t version;
    uint256 prev_hash;
    uint256 merkle_root;
    uint64_t timestamp;
    uint32_t bits;           // PoW difficulty
    uint32_t nonce;
    std::vector<Transaction> txns;
    AIValidation ai_sig;     // AI governance signature
    PoESCertificate poes;    // PoES proof
};

// UTXO model with accounts
class UTXOSet {
    LevelDB db;
    std::unordered_map<uint256, CTxOut> utxos;
    void add(const CTransaction& tx);
    void spend(const COutPoint& outpoint);
};
```

#### 4. Quantum Crypto Module
Located in: `src/crypto/quantum_crypto.cpp`

- **Hash**: SHA-3 (Keccak-256)
- **Signatures**: SPHINCS+ (post-quantum stateless hash-based)
- **Key Exchange**: NewHope (lattice-based)
- **Entropy**: Hardware RNG + system entropy pool

#### 5. JavaScript VM
Embedded **Duktape 2.7.0** with custom bindings:

```javascript
// Example smart contract
contract {
    function transfer(from, to, amount) {
        // Access blockchain state
        const fromBalance = state.getBalance(from);
        if (fromBalance < amount) {
            throw "Insufficient funds";
        }
        state.setBalance(from, fromBalance - amount);
        state.setBalance(to, state.getBalance(to) + amount);
        emit("Transfer", from, to, amount);
    }
}
```

## Bindings Layer (nfx-bindings)

Exposes core functionality via **C ABI** for FFI (Foreign Function Interface):

```c
// C API (nfx_bindings.h)
typedef struct {
    nfx_node_t *node;
    nxf_error_t last_error;
} nfx_context_t;

// Initialize
nfx_context_t* nfx_create(void);
void nfx_init(nfx_context_t *ctx, const char *config_path);

// Blockchain operations
nfx_error_t nfx_get_balance(nfx_context_t *ctx, const char *address, uint64_t *balance);
nfx_error_t nfx_send_transaction(nfx_context_t *ctx, const char *from, const char *to, uint64_t amount);

// Block creation
nfx_error_t nfx_create_block(nfx_context_t *ctx);
nfx_error_t nfx_validate_block(nfx_context_t *ctx, const uint8_t *block_data, size_t len);

// AI governance
nfx_error_t nfx_ai_validate(nfx_context_t *ctx, const uint8_t *block_hash);

// Cleanup
void nfx_destroy(nfx_context_t *ctx);
```

### Memory Management Rules
- Caller allocates output buffers
- Callee returns error codes, not strings
- All strings are null-terminated UTF-8
- Use `nfx_free_string()` for returned heap strings

## Go API (nfx-go)

### Structure
```
nfx-go/
â”œâ”€â”€ api/           # REST API client & server
â”œâ”€â”€ cli/           # Command-line tools
â”œâ”€â”€ rpc/           # JSON-RPC implementation
â”œâ”€â”€ sdk/           # Go SDK library
â””â”€â”€ wallet/        # Wallet implementation (optional)
```

### Main Components

#### 1. RPC Client
```go
type Client struct {
    baseURL    string
    username   string
    password   string
    httpClient *http.Client
}

func (c *Client) GetBalance(address string) (uint64, error)
func (c *Client) SendTransaction(tx *Transaction) (string, error)
func (c *Client) GetBlock(hash string) (*Block, error)
func (c *Client) GetMempool() ([]*Transaction, error)
```

#### 2. REST API Server
```go
// Endpoints
POST   /api/v1/transactions        # Create transaction
GET    /api/v1/balance/:address    # Get balance
GET    /api/v1/blocks/:hash        # Get block
POST   /api/v1/blocks              # Submit block
GET    /api/v1/status              # Node status
```

#### 3. CLI Tool
```bash
# Commands
nfx-cli balance <address>
nfx-cli send --from=<addr> --to=<addr> --amount=<nfx>
nfx-cli block create
nfx-cli status
```

## Wallet Module (nfx-wallet)

### Architecture
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚           QML Frontend Layer                â”‚
â”‚  (Views: Overview, Send, Receive, Staking) â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚            C++ Backend Layer                â”‚
â”‚  (WalletManager, NFXAPI, NodeDiscovery)    â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚          Go/CFunctions Layer                â”‚
â”‚  (RPC calls, transaction signing)          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Key Classes

#### `NFXWallet` (main.cpp)
- Application lifecycle management
- Config loading/saving
- Single instance enforcement

#### `WalletManager` (walletmanager.cpp)
- Wallet database (SQLite)
- Key management (BIP-39 mnemonic)
- Transaction history
- Address book

#### `NFXAPI` (nfxapi.cpp)
- RPC communication with node
- Transaction broadcasting
- Balance queries
- Block/transaction fetches

#### `NodeDiscovery` (nodediscovery.cpp)
- Local node detection (localhost)
- Remote node list from peers
- Manual node addition

### UI Components (QML)

- `MainView.qml` - Main window with tab navigation
- `OverviewPage.qml` - Balance, recent transactions
- `SendCoinsDialog.qml` - Send form with address validator
- `ReceivePage.qml` - QR code generation for receiving
- `StakingPage.qml` - Stake management and rewards
- `GuardiansPage.qml` - Guardian node monitoring
- `IslandsPage.qml` - Island/ecosystem stats

## Data Flow Examples

### 1. Sending a Transaction

```
Wallet UI (QML)
    â”‚
    â–¼
WalletManager::createTransaction()
    â”‚
    â”œâ”€â–º Generate keypair (if needed)
    â”œâ”€â–º Build transaction (inputs, outputs)
    â”œâ”€â–º Sign with private key (ECDSA/SPHINCS+)
    â””â”€â–º Call NFXAPI::sendTransaction()
            â”‚
            â–¼
        RPC POST /api/v1/transactions
            â”‚
            â–¼
        Go API Server (nfx-go)
            â”‚
            â–¼
        C Bindings (CGo)
            â”‚
            â–¼
        nfx-core::Mempool::add()
            â”‚
            â–¼
        P2P Broadcast to peers
```

### 2. Mining / Block Creation

```
PoESValidator::selectMinter()
    â”‚
    â”œâ”€â–º Check eligible stakers
    â”œâ”€â–º Compute PoES certificate
    â”œâ”€â–º Select based on stake weight
    â”‚
    â–¼
Miner::mineBlock()
    â”‚
    â”œâ”€â–º Collect transactions from mempool
    â”œâ”€â–º Build Merkle root
    â”œâ”€â–º Run AI validation on candidate block
    â”œâ”€â–º Apply PoW (if enabled) or skip
    â”œâ”€â–º Sign with miner's key
    â””â”€â–º Submit via submitBlock()
            â”‚
            â–¼
        ChainState::connectBlock()
            â”‚
            â”œâ”€â–º Validate all transactions
            â”œâ”€â–º Update UTXO set
            â”œâ”€â–º Update account balances
            â”œâ”€â–º Execute smart contracts (VM)
            â””â”€â–º Persist to LevelDB
```

### 3. AI Governance Process

```
Block Proposed
    â”‚
    â–¼
BlockParse â†’ Extract features
    â”‚
    â”œâ”€â–º Transaction counts
    â”œâ”€â–º Timestamp variance
    â”œâ”€â–º Merkle path validity
    â”œâ”€â–º Script sig verification
    â””â”€â–º Fee statistics
    â”‚
    â–¼
NeuralNetwork::forward(features)
    â”‚
    â”œâ”€â–º Input layer: 256 neurons
    â”œâ”€â–º Hidden layers: 2Ã—512 neurons (ReLU)
    â””â”€â–º Output: [valid, suspicious, malicious]
    â”‚
    â–¼
AIScore: valid=0.93, suspicious=0.05, malicious=0.02
    â”‚
    â”œâ”€â–º if valid > 0.8 â†’ accept
    â”œâ”€â–º if suspicious > 0.3 â†’ flag for review
    â””â”€â–º if malicious > 0.5 â†’ reject + ban peer
```

## Network Protocol

### Message Format (simplified)
```c
struct NFXMessage {
    uint32_t magic;          // 0x4E465852 (NFXR)
    uint8_t  command[12];    // "block", "tx", etc.
    uint32_t payload_len;
    uint8_t  checksum[4];    // SHA-256 hash of payload
    uint8_t  payload[];
};
```

### Connection Handshake
```
1. Client â†’ Server: VERSION (protocol version, user agent, best block hash)
2. Server â†’ Client: VERSION (echo client's version)
3. Client â†’ Server: VERACK
4. Server â†’ Client: VERACK
5. Connected! Exchange ADDR messages.
```

### Block Propagation (Greedy Digest Routing)
- Instead of sending full blocks to all peers, send **inventory vectors** (hashes)
- Peers request missing blocks via `GET_DATA`
- Reduces bandwidth by ~70%

## Security Model

### Threat Vectors & Mitigations

| Threat | Mitigation |
|--------|------------|
| 51% Attack | PoES makes it exponentially harder as network grows |
| Quantum Attack | SHA-3, SPHINCS+ signatures resistant to quantum algorithms |
| Sybil Attack | Stake + reputation gating for hypernode status |
| Eclipse Attack | Diverse peer selection, anchor nodes |
| Double Spend | PoES finality: 6 confirmations â‰ˆ 99.999% final |
| Smart Contract Bugs | Sandboxed VM, gas limits, audit tools |

### Node Types

| Type | Role | Requirements |
|------|------|-------------|
| **Full Node** | Stores entire chain, validates all | 100GB+ storage, always online |
| **Hypernode** | Analytics + AI governance | 10,000 NFX stake, high uptime |
| **Light Node** | SPV validation only | Mobile/low-end devices |
| **Archive Node** | Full + historical states | 500GB+ storage, indexers |

## Performance Characteristics

| Metric | Target | Current (v0.8) |
|--------|--------|----------------|
| Block Time | 60 seconds | 60s |
| TPS (theoretical) | 1,000 | ~200 |
| TPS (with sharding) | 10,000+ | N/A |
| Confirmation Finality | 6 blocks (6 min) | 6 blocks |
| Network Latency | <2s global | ~3-5s |
| Wallet Sync Time | <2 min (SPV) | ~90s |
| Block Size | 2 MB | 1 MB |

## Configuration System

All configuration uses **TOML** format. See `config.toml`.

### Key Settings
```toml
[consensus]
consensus = "poes"          # pow|pos|poes
powdifficulty = 1           # PoW difficulty target
posminstake = 1000          # Minimum PoS stake (NFX)

[network]
p2p_port = 8333             # P2P port
rpc_port = 8332             # RPC port
maxconnections = 50         # Peer connections

[ai]
enabled = true              # Enable AI governance
model_path = "/models/"     # Neural network weights
threshold_valid = 0.8       # AI confidence threshold

[security]
max_mempool = 300           # Max mempool MB
max_parallel = 4            # Parallel script threads
```

## Deployment Topologies

### Single Node (Development)
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   nfx-wallet       â”‚
â”‚   nfx-go (RPC)     â”‚
â”‚   nfx-core (all)   â”‚
â”‚   LevelDB storage  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Multi-Node (Production)
```
[Wallet]    [Wallet]    [Wallet]
     â”‚           â”‚           â”‚
     â–¼           â–¼           â–¼
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚   Load Balancer (RPC)      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              â”‚
    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
    â–¼         â–¼         â–¼
â”Œâ”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”
â”‚API1â”‚  â”‚API2â”‚  â”‚API3â”‚ (nfx-go)
â””â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”˜
    â”‚         â”‚         â”‚
    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              â–¼
    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
    â”‚  Hypernode Cluster  â”‚ (nfx-core)
    â”‚  (8-12 geo-dist)    â”‚
    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              â”‚
    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
    â–¼                   â–¼
LevelDB1           LevelDB2 (replicas)
```

## Future Extensions

1. **Sharding**: Horizontal partition of state
2. **Layer-2**: Lightning Network-style payment channels
3. **Zero-Knowledge Proofs**: zk-SNARKs for privacy
4. **Cross-chain Bridges**: Atomic swaps with Bitcoin/Ethereum
5. **Mobile Light Clients**: SPV in iOS/Android apps
6. **IoT Integration**: ARM64 support for Raspberry Pi clusters

---

*See also: [Module Documentation](docs/modules/) | [API Reference](docs/api/) | [Examples](docs/examples/)*
