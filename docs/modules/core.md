# nfx-core Module Documentation

**Module:** `nfx-core`  
**Language:** C++17 (Qt5)  
**Type:** Static/Shared Library + Executable  
**Path:** `nfx-core/`  

## Overview

`nfx-core` is the heart of the NFX Chain â€” a high-performance C++ library implementing the entire blockchain logic. It provides:

- Consensus engine (PoW/PoS/PoES)
- P2P networking with hypernode topology
- Quantum-resistant cryptography (SHA-3, SPHINCS+)
- LevelDB-based persistent storage
- JavaScript smart contract VM (Duktape)
- JSON-RPC server
- Daemon process management

## Directory Structure

```
nfx-core/
â”œâ”€â”€ CMakeLists.txt          # Build configuration
â”œâ”€â”€ README.md
â”œâ”€â”€ include/
â”‚   â””â”€â”€ nfx/               # Public API headers
â”‚       â”œâ”€â”€ nfx.hpp
â”‚       â”œâ”€â”€ types.hpp
â”‚       â”œâ”€â”€ consensus/
â”‚       â”œâ”€â”€ network/
â”‚       â”œâ”€â”€ blockchain/
â”‚       â””â”€â”€ crypto/
â”œâ”€â”€ src/
â”‚   â”œâ”€â”€ main.cpp           # nfx-node entry point
â”‚   â”œâ”€â”€ main_daemon.cpp    # Daemon mode entry
â”‚   â”œâ”€â”€ daemon.cpp/h       # Daemon lifecycle
â”‚   â”œâ”€â”€ nfx.hpp           # Internal core header
â”‚   â”œâ”€â”€ cgo_bindings.cpp  # C ABI exports
â”‚   â”œâ”€â”€ cgo_bindings.hpp
â”‚   â”œâ”€â”€ CMakeLists.txt    # Subdirectory build
â”‚   â”‚
â”‚   â”œâ”€â”€ consensus/        # Consensus protocols
â”‚   â”‚   â”œâ”€â”€ expansive_island_minter.cpp/h  # PoES implementation
â”‚   â”‚   â”œâ”€â”€ physical_guardian.cpp/h        # Guardian node logic
â”‚   â”‚   â”œâ”€â”€ poes_calculator.cpp/h          # PoES math
â”‚   â”‚   â”œâ”€â”€ pos_transition_manager.cpp/h   # PoS management
â”‚   â”‚   â”œâ”€â”€ reward_manager.cpp/h           # Rewards distribution
â”‚   â”‚   â”œâ”€â”€ stake_delegator.cpp/h          # Stake delegation
â”‚   â”‚   â””â”€â”€ state_manager.cpp/h            # Consensus state
â”‚   â”‚
â”‚   â”œâ”€â”€ network/          # P2P networking
â”‚   â”œâ”€â”€ p2p/              # Peer-to-peer protocol
â”‚   â”œâ”€â”€ rpc/              # JSON-RPC server
â”‚   â”œâ”€â”€ blockchain/       # Block/transaction/UTXO
â”‚   â”œâ”€â”€ storage/          # LevelDB wrappers
â”‚   â”œâ”€â”€ crypto/           # Quantum-safe crypto
â”‚   â”œâ”€â”€ vm/               # JavaScript VM
â”‚   â”œâ”€â”€ ai/               # AI governance
â”‚   â””â”€â”€ tests/            # Unit & integration tests
â””â”€â”€ tests/                # Build system tests
```

## Building

### As Library (for linking)

```cmake
# In your CMakeLists.txt
find_package(nfx-core REQUIRED)
add_executable(myapp main.cpp)
target_link_libraries(myapp nfx-core)
```

### As Standalone Binary

```bash
cd nfx-core
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

# Output files:
# - nfx-node      (main executable)
# - libnfx-core.so (shared library, if BUILD_SHARED_LIBS=ON)
```

### CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `CMAKE_BUILD_TYPE` | `Release` | `Debug`, `Release`, `RelWithDebInfo` |
| `BUILD_SHARED_LIBS` | `OFF` | Build shared library (.so/.dll) |
| `ENABLE_TESTS` | `ON` | Build test executables |
| `ENABLE_AI` | `ON` | Enable AI governance module |
| `ENABLE_QUANTUM_CRYPTO` | `ON` | Enable SHA-3/SPHINCS+ |

Example:
```bash
cmake .. \
    -DCMAKE_BUILD_TYPE=Debug \
    -DENABLE_TESTS=ON \
    -DENABLE_AI=ON \
    -DBUILD_SHARED_LIBS=ON
```

## Core Components

### 1. Blockchain Layer

**Files:** `src/blockchain/`

**Key Classes:**

```cpp
// Block structure
class Block {
public:
    uint32_t version;
    uint256 hashPrevBlock;
    uint256 hashMerkleRoot;
    uint64_t nTime;
    uint32_t nBits;
    uint32_t nNonce;
    std::vector<CTransaction> vtx;
    AIValidation ai_sig;      // AI governance signature
    PoESProof poes_proof;     // PoES certificate

    uint256 GetHash() const;
    bool CheckProofOfWork() const;
    bool Connect(Chainstate& state);
};

// Transaction
class CTransaction {
public:
    int32_t nVersion;
    std::vector<CTxIn> vin;
    std::vector<CTxOut> vout;
    uint32_t nLockTime;
    std::vector<uint8_t> scriptWitness;  // SegWit

    uint256 GetHash() const;
    uint256 GetWitnessHash() const;
    bool IsCoinBase() const;
};

// Unspent output
class COutPoint {
    uint256 hash;
    uint32_t n;
};

class CTxOut {
    CAmount nValue;
    CScript scriptPubKey;
};
```

**UTXO Management:**

`src/blockchain/utxo.cpp` maintains the set of unspent outputs:

```cpp
class UTXOSet {
private:
    LevelDB db;
    std::unordered_map<uint256, CTxOut> mapUTXO;

public:
    bool Add(const CTransaction& tx);
    bool Spend(const COutPoint& outpoint);
    bool GetUTXO(const COutPoint& outpoint, CTxOut& txout);
    size_t Count() const;
    void Flush();  // Persist to disk
};
```

### 2. Consensus Layer

**Files:** `src/consensus/`

#### Proof of Exponential Security (PoES)

The PoES algorithm is implemented in `expansive_island_minter.cpp`:

```cpp
class PoESValidator {
public:
    // Calculate security probability
    static double CalculateSecurityFactor(
        size_t honest_nodes,
        size_t total_nodes,
        int64_t blocks_observed
    );

    // Select next block minter
    static std::vector<uint160> SelectMinters(
        const std::vector<StakeEntry>& stakes,
        uint32_t block_height
    );

    // Verify PoES certificate
    static bool VerifyCertificate(
        const PoESProof& proof,
        const Block& block
    );
};
```

**Security Formula:**
```
S(t) = 1 - e^(-kÂ·hÂ·t)

k: security constant (adjustable)
h: hash power ratio (honest/total)
t: number of blocks observed

An attacker with < 51% honest hash power will exponentially
lose probability of creating longest chain over time.
```

#### Block Validation Pipeline

```cpp
enum class BlockValidationResult {
    VALID,
    INVALID_PROOF_OF_WORK,
    INVALID_MERKLE_ROOT,
    INVALID_SIGNATURES,
    AI_REJECTED,          // AI governance rejected
    POES_INVALID,
    INVALID_STATE,
    DUPLICATE_BLOCK
};

class BlockValidator {
public:
    BlockValidationResult ValidateBlock(
        const CBlock& block,
        const ChainState& state
    ) {
        // 1. Proof of Work check
        if (!CheckProofOfWork(block)) {
            return INVALID_PROOF_OF_WORK;
        }

        // 2. Merkle root validation
        if (!VerifyMerkleRoot(block)) {
            return INVALID_MERKLE_ROOT;
        }

        // 3. AI governance validation (if enabled)
        if (config.ai.enabled) {
            double score = ai_engine_->ValidateBlock(block);
            if (score < config.ai.threshold_valid) {
                return AI_REJECTED;
            }
        }

        // 4. PoES verification (PoES mode)
        if (config.consensus == "poes") {
            if (!poes_validator_.VerifyCertificate(
                block.poes_proof,
                block.hashPrevBlock
            )) {
                return POES_INVALID;
            }
        }

        // 5. Transaction validity
        for (const auto& tx : block.vtx) {
            if (!ValidateTransaction(tx, state)) {
                return INVALID_TRANSACTION;
            }
        }

        return VALID;
    }
};
```

### 3. P2P Network Layer

**Files:** `src/p2p/`, `src/network/`

#### Node Class

```cpp
class Node {
public:
    Node(const NodeConfig& cfg);
    ~Node();

    // Start/stop
    bool Start();
    void Stop();

    // Peer management
    void AddPeer(const std::string& addr);
    void RemovePeer(const NodeId& id);
    std::vector<PeerInfo> GetPeers() const;

    // Message handlers
    void HandleMessage(const NetMsg& msg, Peer* from);
    void Broadcast(const NetMsg& msg);
    void SendTo(const NodeId& id, const NetMsg& msg);

    // State
    bool IsConnected() const;
    size_t PeerCount() const;

private:
    std::unique_ptr<P2PProtocol> protocol_;
    std::unique_ptr<PeerManager> peers_;
    std::unique_ptr<MessageDispatcher> dispatcher_;
    bool running_;
};
```

#### Hypernode Connectivity

Hypernodes are specially-staked full nodes that serve as network hubs:

```cpp
class Hypernode : public Node {
public:
    Hypernode(const HypernodeConfig& cfg);
    ~Hypernode();

    // Register as hypernode (requires stake)
    bool RegisterStake(const std::string& txid);
    void PublishAnalytics(const NetworkAnalytics& data);

    // AI governance participation
    void SubmitAIVote(const BlockHash& hash, AIVote vote);
    AIVote GetAIVote(const BlockHash& hash);

private:
    StakeEntry stake_;
    AIVoter ai_voter_;
    AnalyticsPublisher analytics_;
};
```

**Hypernode Selection Algorithm:**
```cpp
std::vector<Hypernode*> SelectHypernodes(
    const std::vector<Hypernode*>& candidates,
    size_t count
) {
    // Sort by: stake * uptime * geographic_diversity
    std::sort(candidates.begin(), candidates.end(),
        [](const Hypernode* a, const Hypernode* b) {
            double score_a = a->GetStake() * a->GetUptime() *
                            a->GetGeoScore();
            double score_b = b->GetStake() * b->GetUptime() *
                            b->GetGeoScore();
            return score_a > score_b;
        });

    return std::vector<Hypernode>(
        candidates.begin(),
        candidates.begin() + std::min(count, candidates.size())
    );
}
```

### 4. Cryptography Module

**Files:** `src/crypto/`, `src/crypto/CMakeLists.txt`

#### Quantum-Resistant Algorithms

```cpp
namespace crypto {

// SHA-3 (Keccak) 256-bit hash
std::array<uint8_t, 32> SHA3_256(
    const std::vector<uint8_t>& data
);

// SPHINCS+ signature (post-quantum)
class SphincsPlus {
public:
    SphincsPlus(const std::string& params = "sphincs-shake256-128f-simple");

    std::pair<std::vector<uint8_t>, std::vector<uint8_t>>
    GenerateKeypair();

    std::vector<uint8_t> Sign(
        const std::vector<uint8_t>& msg,
        const std::vector<uint8_t>& privkey
    );

    bool Verify(
        const std::vector<uint8_t>& msg,
        const std::vector<uint8_t>& sig,
        const std::vector<uint8_t>& pubkey
    );
};

// NewHope key exchange (lattice-based)
class NewHope {
public:
    std::pair<std::vector<uint8_t>, std::vector<uint8_t>>
    GenerateKeypair();

    std::vector<uint8_t> Encapsulate(
        const std::vector<uint8_t>& pubkey,
        std::vector<uint8_t>& shared_secret
    );

    bool Decapsulate(
        const std::vector<uint8_t>& ciphertext,
        const std::vector<uint8_t>& privkey,
        std::vector<uint8_t>& shared_secret
    );
};

} // namespace crypto
```

**Usage Example:**

```cpp
// Generate wallet address with post-quantum keys
#include <nfx/crypto/quantum_crypto.hpp>

Keypair keypair = crypto::GenerateKeyPair();
std::string address = crypto::AddressFromPubKey(keypair.pubkey);

// Sign transaction
std::vector<uint8_t> sig = crypto::Sign(
    tx_hash,
    keypair.privkey
);
```

#### Key Derivation (BIP-39 + BIP-32)

```cpp
#include <nfx/crypto/hd_wallet.hpp>

HDWallet wallet;
wallet = HDWallet::FromMnemonic(
    "abandon abandon abandon ..."  // 12-word seed
);

// Derive account 0, external chain, address 0
Key key = wallet.Derive(
    "m/44'/777'/0'/0/0"  // BIP-44 for NFX Chain
);

std::string address = key.GetAddress();
std::string privkey = key.GetWIF();  // Wallet Import Format
```

### 5. Virtual Machine (JavaScript)

**Files:** `src/vm/`

NFX uses **Duktape 2.7.0** embedded VM with custom bindings:

#### VM Context

```cpp
class ScriptVM {
public:
    ScriptVM();
    ~ScriptVM();

    // Execute script
    JSValue Execute(
        const std::string& source,
        const std::string& filename = "contract.js"
    );

    // Bind C++ functions to JS
    void RegisterFunction(
        const std::string& name,
        std::function<JSValue(JSValue*)> func
    );

    // Expose blockchain state
    void SetState(const ChainState& state);
    ChainState GetState() const;

    // Gas metering
    void SetGasLimit(uint64_t gas);
    uint64_t GetGasUsed() const;
    bool OutOfGas() const;

private:
    duk_context* ctx_;
    ChainState* state_;
    uint64_t gas_used_;
};
```

#### Built-in JS API

Contracts have access to:

```javascript
// State manipulation
state.setBalance(address, amount);
state.getBalance(address);

// Event emission
emit("Transfer", from, to, amount);
emit("Stake", validator, amount);

// Blockchain access
blockchain.height;
blockchain.difficulty;
blockchain.timestamp;

// Validation
require(condition, "Error message");
assert(expression);

// Logging
log("Debug message", level);
```

**Example Contract:**

```javascript
// Simple token contract
contract NFXToken {
    mapping balances;
    mapping allowances;

    constructor(initialSupply) {
        balances[msg.sender] = initialSupply;
    }

    function transfer(to, amount) {
        require(balances[msg.sender] >= amount);
        balances[msg.sender] -= amount;
        balances[to] += amount;
        emit("Transfer", msg.sender, to, amount);
    }

    function approve(spender, amount) {
        allowances[msg.sender][spender] = amount;
        emit("Approval", msg.sender, spender, amount);
    }

    function transferFrom(from, to, amount) {
        require(balances[from] >= amount);
        require(allowances[from][msg.sender] >= amount);
        balances[from] -= amount;
        allowances[from][msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

### 6. Storage Layer

**Files:** `src/storage/`

LevelDB wrappers for chain state:

```cpp
class LevelDBStore {
public:
    LevelDBStore(const std::string& datadir);
    ~LevelDBStore();

    // Generic key-value
    bool Put(const std::string& key, const Slice& value);
    bool Get(const std::string& key, std::string* value) const;
    bool Delete(const std::string& key);
    Iterator* NewIterator();

    // Chain-specific
    bool PutBlock(const CBlock& block);
    bool GetBlock(const BlockHash& hash, CBlock& block);
    bool PutUTXO(const COutPoint& outpoint, const CTxOut& txout);
    bool GetUTXO(const COutPoint& outpoint, CTxOut& txout);

    // State
    size_t ApproximateSize() const;
    void Compact();

private:
    leveldb::DB* db_;
    leveldb::Options opts_;
    leveldb::ReadOptions ropts_;
    leveldb::WriteOptions wopts_;
};
```

**ChainState** maintains system state in memory, persisted to LevelDB:

```cpp
class ChainState {
public:
    // Blockchain navigation
    CBlockIndex* GetIndex(const BlockHash& hash) const;
    CBlockIndex* GetTip() const;
    void SetTip(CBlockIndex* tip);

    // UTXO queries
    bool GetUTXO(const COutPoint& outpoint, CTxOut& txout);
    void AddUTXO(const COutPoint& outpoint, const CTxOut& txout);
    void SpendUTXO(const COutPoint& outpoint);

    // Balance queries (account model)
    uint64_t GetBalance(const std::string& address) const;
    void AddBalance(const std::string& address, CAmount amount);

    // Nonce tracking
    uint64_t GetNonce(const std::string& address) const;
    void SetNonce(const std::string& address, uint64_t nonce);

    // Persistence
    void Flush();          // Write to LevelDB
    void LoadFromDisk();   // Load from LevelDB

private:
    std::unique_ptr<LevelDBStore> store_;
    std::unordered_map<uint256, std::unique_ptr<CBlockIndex>> block_index_;
    std::unordered_map<COutPoint, CTxOut> utxo_set_;
    std::unordered_map<std::string, uint64_t> balances_;
    CBlockIndex* tip_;
};
```

### 7. AI Governance Module

**Files:** `src/ai/`

#### Neural Network Classifier

```cpp
class AIGovernanceEngine {
public:
    AIGovernanceEngine(const std::string& model_path);
    ~AIGovernanceEngine();

    // Evaluate block for anomalies
    AIScore EvaluateBlock(const CBlock& block);

    // Train model (offline, admin only)
    void Train(
        const std::vector<LabeledBlock>& dataset,
        size_t epochs = 100
    );

    // Save/load model weights
    bool SaveModel(const std::string& path);
    bool LoadModel(const std::string& path);

private:
    // Neural network (using tiny-dnn or custom)
    struct Network {
        // Input layer: 256 features
        std::vector<float> input;

        // Hidden layers: 2 Ã— 512 neurons (ReLU)
        std::vector<std::vector<float>> hidden1;
        std::vector<std::vector<float>> hidden2;

        // Output: [valid, suspicious, malicious]
        std::vector<float> output;

        // Weights & biases (loaded from file)
        std::vector<float> w1, b1, w2, b2, w3, b3;
    };

    std::unique_ptr<Network> net_;
    FeatureExtractor extractor_;
};
```

**Feature Extraction:**

```cpp
class FeatureExtractor {
public:
    std::vector<float> Extract(const CBlock& block) {
        std::vector<float> features;

        // 1. Transaction count
        features.push_back(block.vtx.size());

        // 2. Block size in bytes
        features.push_back(GetSerializeSize(block));

        // 3. Timestamp variance from median
        features.push_back(ComputeTimeVariance(block));

        // 4. Fee distribution entropy
        features.push_back(ComputeFeeEntropy(block));

        // 5. Script types (P2PKH, P2SH, SegWit, etc.)
        auto type_counts = CountScriptTypes(block);
        for (auto& [type, count] : type_counts) {
            features.push_back(count);
        }

        // 6. Duplicate transaction detection
        features.push_back(CountDuplicates(block));

        // 7. Merkle path validity (precomputed)
        features.push_back(block.merkle_valid ? 1.0f : 0.0f);

        // 8. Input/output ratio
        features.push_back(ComputeInputOutputRatio(block));

        // ... up to 256 total features

        return Normalize(features);  // Z-score normalization
    }
};
```

**Decision Logic:**

```cpp
AIScore AIGovernanceEngine::EvaluateBlock(const CBlock& block) {
    auto features = extractor_.Extract(block);
    auto predictions = net_->Forward(features);

    AIScore score;
    score.valid = predictions[0];
    score.suspicious = predictions[1];
    score.malicious = predictions[2];

    // Action based on thresholds
    if (score.malicious > config.threshold_malicious) {
        score.action = AIAction::REJECT;
        score.reason = "Malicious block pattern detected";
    } else if (score.suspicious > config.threshold_suspicious) {
        score.action = AIAction::FLAG;
        score.reason = "Suspicious characteristics";
    } else {
        score.action = AIAction::ACCEPT;
    }

    return score;
}
```

### 8. RPC Server

**Files:** `src/rpc/`

JSON-RPC interface using **libmicrohttpd** or **Qt HTTP server**:

```cpp
class RPCServer {
public:
    RPCServer(const RPCConfig& cfg, ChainState& state);
    ~RPCServer();

    bool Start();
    void Stop();

private:
    void HandleRequest(
        MHD_Connection* conn,
        const std::string& method,
        const json& params
    );

    // RPC methods
    json GetBlockchainInfo();
    json GetBlock(const std::string& hash);
    json GetBlockCount();
    json GetBalance(const std::string& address);
    json SendTransaction(const json& tx);
    json ListUnspent();

    // Wallet RPC (if wallet enabled)
    json WalletPassphrase(const std::string& passphrase);
    json WalletCreateAddress();

    Server server_;
    ChainState& state_;
    std::unique_ptr<WalletService> wallet_;
};
```

**Available RPC Methods:**

| Method | Params | Returns | Description |
|--------|--------|---------|-------------|
| `getblockchaininfo` | none | object | Chain info |
| `getblockcount` | none | number | Current height |
| `getblock` | hash, verbose? | object | Block data |
| `gettransaction` | txid | object | Transaction details |
| `getbalance` | address | number | Account balance |
| `sendtransaction` | from, to, amount | string | TXID |
| `getmempoolinfo` | none | object | Mempool stats |
| `getpeerinfo` | none | array | Connected peers |

**Example RPC Call:**

```bash
curl --user nfxuser:mypassword \
     -X POST http://localhost:8332/ \
     -H "Content-Type: application/json" \
     -d '{
       "jsonrpc":"1.0",
       "id":"test",
       "method":"getbalance",
       "params":["NFX123..."]
     }'
```

### 9. Daemon Process

**Files:** `src/daemon.cpp`, `src/main_daemon.cpp`

```cpp
class Daemon {
public:
    Daemon(const DaemonConfig& cfg);
    ~Daemon();

    int Run();

private:
    void SignalHandler(int sig);
    void PIDFileCreate(const std::string& path);
    void PIDFileRemove();

    bool Init();
    void StartRPC();
    void StartP2P();
    void StartAIService();

    void MainLoop();
    void Shutdown();

    std::unique_ptr<ChainState> state_;
    std::unique_ptr<Node> p2p_node_;
    std::unique_ptr<RPCServer> rpc_server_;
    std::unique_ptr<AIGovernanceEngine> ai_engine_;

    bool running_ = false;
    std::string pid_file_;
};
```

**Daemon Commands:**

```bash
# Start daemon
nfx-node --daemon --config config.toml

# Stop daemon
nfx-cli stop

# Get status
nfx-cli getinfo

# View logs
tail -f ~/.nfx/logs/nfx.log

# Debug mode (foreground)
nfx-node --console --log-level=debug
```

### 10. Testing Infrastructure

**Files:** `src/test_*.cpp`, `tests/`

#### Unit Tests (Google Test)

```cpp
#include <gtest/gtest.h>

TEST(BlockTest, BlockHashConsistency) {
    CBlock block = CreateTestBlock(1);
    uint256 hash1 = block.GetHash();
    uint256 hash2 = block.GetHash();
    ASSERT_EQ(hash1, hash2);
}

TEST(ConsensusTest, PoESValidation) {
    PoESValidator validator;
    PoESProof proof = GenerateTestProof();

    bool valid = validator.VerifyCertificate(proof, test_block);
    EXPECT_TRUE(valid);
}
```

Run tests:
```bash
cd nfx-core/build
ctest            # All tests
ctest -R PoES    # Filter by regex
./test_poes     # Direct binary
```

#### Integration Tests

Located in `tests/`:
```bash
# Run full integration test
cd tests
./run_integration_tests.sh

# Simulate 100 blocks
./simulate_chain --blocks=100 --peers=5
```

## Development Guidelines

### Code Style

- **Indentation**: 4 spaces (no tabs)
- **Line length**: â‰¤ 120 characters
- **Braces**: K&R style
- **Naming**: `CamelCase` for types, `snake_case` for functions/variables

**Example:**
```cpp
// Good
class BlockValidator {
public:
    bool ValidateBlock(const CBlock& block);
private:
    ChainState& state_;
};

// Bad
class block_validator {
public:
    bool validateBlock(CBlock block);
private:
    ChainState& State;
};
```

### Adding a New RPC Method

1. Declare in `src/rpc/rpc_server.h`:
```cpp
json GetMyNewMethod(const std::string& param);
```

2. Implement in `src/rpc/rpc_server.cpp`:
```cpp
json RPCServer::GetMyNewMethod(const std::string& param) {
    // Validate input
    if (param.empty()) {
        throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid parameter");
    }

    // Do work
    auto result = state_->ComputeSomething(param);

    return json{{"result", result}};
}
```

3. Register in constructor:
```cpp
handlers_["mynewmethod"] = [this](const json& params) {
    return GetMyNewMethod(params[0].get<std::string>());
};
```

4. Add to documentation in `docs/api/rpc.md`

### Memory Management

- Use `std::unique_ptr` for automatic cleanup
- Raw pointers only for non-owning references
- Never use `new`/`delete` directly; use `make_unique<T>()`

**Example:**
```cpp
// Good
auto node = std::make_unique<Node>(config);
node->Start();

// Bad
Node* node = new Node(config);
node->Start();
delete node;  // forget? leak!
```

### Thread Safety

- `ChainState` is **not** thread-safe â†’ protect with mutex
- `UTXOSet` modifications require exclusive lock
- RPC handlers run on thread pool

```cpp
class ChainState {
    mutable std::shared_mutex mutex_;

public:
    uint64_t GetBalance(const std::string& addr) const {
        std::shared_lock lock(mutex_);
        return balances_.at(addr);
    }

    void SetBalance(const std::string& addr, uint64_t amount) {
        std::unique_lock lock(mutex_);
        balances_[addr] = amount;
    }
};
```

## Performance Tips

### 1. Build with LTO (Link-Time Optimization)

```bash
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON
make -j$(nproc)
```

### 2. Enable BTI (Branch Target Identification)

```bash
# GCC 8+ on x86_64
cmake .. -DCMAKE_CXX_FLAGS="-mbranch-protection=standard"
```

### 3. Tune LevelDB Options

```cpp
// In storage initialization
leveldb::Options opts;
opts.compression = leveldb::kSnappyCompression;  // Fast + good compression
opts.paranoid_checks = true;                     // Extra integrity checks
opts.write_buffer_size = 64 * 1024 * 1024;       // 64MB write buffer
opts.max_open_files = 1000;                      // Reduce file handle churn
```

### 4. Use Huge Pages (Linux)

```bash
# Allocate huge pages (requires root)
echo 1024 | sudo tee /proc/sys/vm/nr_hugepages

# Pin process to NUMA node 0
numactl --cpunodebind=0 --membind=0 ./nfx-node
```

### 5. Profile with perf

```bash
# Record profile
perf record -g -- ./nfx-node --testnet

# View flame graph
perf script | stackcollapse-perf.pl | flamegraph.pl > profile.svg
```

## Debugging

### Enable Debug Logging

```bash
./nfx-node --log-level=debug --log-categories="net,consensus,rpc"

# Log to file with rotation
./nfx-node --log-file=/var/log/nfx.log --log-level=info
```

### GDB Debugging

```bash
gdb --args ./nfx-node --testnet --datadir=/tmp/test
(gdb) break consensus/expansive_island_minter.cpp:123
(gdb) run
(gdb) bt   # Backtrace on crash
(gdb) p *block   # Print block object
```

### Valgrind (Memory Leaks)

```bash
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         ./nfx-node --testnet
```

## API Reference

### Public Headers

All public headers are in `include/nfx/`:

| Header | Purpose |
|--------|---------|
| `nfx.hpp` | Main umbrella header (includes all) |
| `types.hpp` | Common types (uint256, CAmount, etc.) |
| `blockchain.hpp` | Block, transaction, UTXO classes |
| `consensus.hpp` | Consensus engine interfaces |
| `network.hpp` | P2P networking classes |
| `crypto.hpp` | Cryptography functions |
| `vm.hpp` | JavaScript VM bindings |
| `rpc.hpp` | JSON-RPC server/client |

### Linking Against nfx-core

```cmake
find_package(nfx REQUIRED)

add_executable(myapp main.cpp)
target_link_libraries(myapp nfx::nfx-core)

# Or pkg-config
pkg_check_modules(NFX REQUIRED nfx-core)
target_include_directories(myapp ${NFX_INCLUDE_DIRS})
target_link_libraries(myapp ${NFX_LIBRARIES})
```

## Contributing

When modifying nfx-core:

1. Run all tests: `make test` or `ctest`
2. Check formatting: `clang-format -i src/*.cpp include/*.hpp`
3. Update documentation in `docs/`
4. Add unit test for new feature
5. Ensure no memory leaks (valgrind)
6. Submit PR with detailed description

---

*Next: [Bindings Module](bindings.md) | [Go API](go.md) | [Wallet](wallet.md)*
