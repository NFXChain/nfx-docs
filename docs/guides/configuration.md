# Configuration Reference

**File:** `config.toml` (TOML format)

Complete reference for all configuration options in NFX Chain node.

## Table of Contents

- [File Locations](#file-locations)
- [Configuration Hierarchy](#configuration-hierarchy)
- [Global Settings](#global-settings)
- [Network](#network)
- [Consensus](#consensus)
- [Cryptography](#cryptography)
- [AI Governance](#ai-governance)
- [Database](#database)
- [Virtual Machine](#virtual-machine)
- [RPC Server](#rpc-server)
- [Wallet](#wallet)
- [Performance](#performance)
- [Logging](#logging)
- [Examples](#examples)

---

## File Locations

NFX searches for config files in this order:

1. Path specified by `--config` flag
2. `$HOME/.nfx/config.toml` (default)
3. `/etc/nfx/config.toml` (system-wide)
4. `./config.toml` (current directory)

**Example:**
```bash
# Use custom config
nfx-node --config /home/user/my-config.toml
```

---

## Configuration Hierarchy

Command-line flags override config file:

```bash
# config.toml sets testnet=true
# But CLI flag overrides:
nfx-node --mainnet   # Uses mainnet despite config

# Priority (highest to lowest):
# 1. CLI flags
# 2. Environment variables (NFX_* prefix)
# 3. Config file
# 4. Defaults
```

---

## Global Settings

### Working Directory & Paths

```toml
# Data directory (blockchain storage)
datadir = "/home/user/.nfx/data"

# Log file path
logfile = "/home/user/.nfx/logs/nfx.log"

# Maximum log file size before rotation (MB)
maxlogfilesize = 50

# Number of rotated log files to keep
maxlogfiles = 10

# PID file location (daemon mode)
pidfile = "/home/user/.nfx/nfx.pid"
```

### Logging

```toml
[logging]
logtimestamps = true      # Include ISO timestamps in logs
logthreadnames = false    # Include thread IDs
logtimeouts = false       # Log slow operations

# Log categories (comma-separated, empty = all)
# Available: net, consensus, rpc, db, vm, ai, wallet, all
logcategories = ["net", "consensus", "rpc"]

# Console logging
console = true           # Log to stdout/stderr
logcolors = true         # Colorized output (if terminal supports)

# Syslog
syslog = false           # Also log to syslog
syslogfacility = "local0"  # Syslog facility
```

### Testing & Development

```toml
[test]
enabled = false          # Enable test mode (no real network)
regtest = false          # Regression test mode (local only)
signet = false           # Signet (custom testnet)
fallbackfee = 0.00001    # Minimum fee (NFX) for non-standard tx
```

---

## Network

### Network Selection

```toml
[network]
# Either testnet or mainnet (exclusive)
testnet = true           # true = testnet, false = mainnet
# OR use explicit network spec
# network = "testnet"

# Ports
p2p_port = 18333         # P2P port (testnet default)
rpc_port = 18332         # RPC port (testnet default)
ws_port = 18334          # WebSocket notifications port
zmq_port = 18335         # ZMQ notification port

# Bind addresses (0.0.0.0 = all interfaces)
p2p_bind = "0.0.0.0:18333"
rpc_bind = "127.0.0.1:18332"  # Only local (recommended for security)

# External IP (if behind NAT)
externalip = "203.0.113.10:18333"

# Maximum peers
maxconnections = 50      # Total connections (inbound + outbound)
maxoutbound = 8          # Max outbound connections
max inbound = 42         # Max inbound connections

# Seed nodes (initial bootstrapping)
dns_seeds = [
    "seed.testnet.nfxchain.io",
    "seed2.testnet.nfxchain.io",
    "seed3.testnet.nfxchain.io"
]

# Fixed peers (add to always connect)
addnode = [
    "192.168.1.10:18333",
    "203.0.113.20:18333"
]

# Banned nodes (will not connect)
banned_nodes = [
    "1.2.3.4:18333"
]

# Connection timeout (seconds)
connection_timeout = 30

# Enable UPnP for port mapping (not recommended for servers)
upnp = false
```

### Network Types Comparison

| Setting | Mainnet | Testnet |
|---------|---------|---------|
| `testnet` | `false` | `true` |
| `p2p_port` | `8333` | `18333` |
| `rpc_port` | `8332` | `18332` |
| `dns_seeds` | `mainnet.nfxchain.io` | `testnet.nfxchain.io` |
| `genesis_block` | Hardcoded mainnet hash | Hardcoded testnet hash |

---

## Consensus

### Consensus Algorithm Selection

```toml
[consensus]
# Choose one: "pow", "pos", "poes"
consensus = "poes"

# Common params
blocktime = 60           # Target time between blocks (seconds)
maxt blocksize = 2000000 # 2 MB max block size
mintxfee = 1000          # Minimum fee in satoshis
maxsigops = 20000        # Max sigops per block (anti-DoS)
```

### Proof of Work (PoW)

```toml
[consensus.pow]
enabled = true             # Enable PoW component
algorithm = "sha3"         # "sha256" or "sha3"
difficulty = 1.0           # Initial difficulty

# Difficulty adjustment parameters
pow_adjust_window = 2016   # Blocks between adjustments (â‰ˆ 2 weeks @ 60s blocktime)
pow_target_spacing = 60    # Target spacing (matches blocktime)
pow_allow_minertips = true # Allow miners to collect tx fees

# Mining requires PoW when `consensus` includes "pow"
```

**Example:** Pure PoW mode:
```toml
[consensus]
consensus = "pow"
[consensus.pow]
enabled = true
```

### Proof of Stake (PoS)

```toml
[consensus.pos]
enabled = true              # Enable PoS component
min_stake = 10000000000     # Minimum stake: 100 NFX (assuming 8 decimals)
stake_threshold = 0.05      # Minimum % of total supply to participate
stake_maturity = 1000       # Blocks until stake matures (â‰ˆ 16 hours)
reward_percent = 0.05       # 5% annual inflation to stakers

# Stake checking
check_stake = true          # Verify stake signatures
stake_combine_dust = true   # Combine dust UTXOs into single stake
```

**Example:** Pure PoS mode:
```toml
[consensus]
consensus = "pos"
[consensus.pos]
enabled = true
min_stake = 10000000000
```

### Proof of Exponential Security (PoES) â€” Default

```toml
[consensus.poes]
enabled = true

# Security factor formula: S(t) = 1 - e^(-kÂ·hÂ·nÂ·t)
# k: security_constant
# h: relative hash power
# n: number of honest nodes
# t: time (blocks)

security_constant = 1.0    # Multiplier (adjustable governance parameter)
min_security_level = 0.95  # Minimum acceptable security threshold

# Validator selection
max_validators = 101       # Max PoES validators per round
min_validators = 7         # Minimum for quorum
validator_rotation = 100   # Blocks before rotating validator set

# AI integration (if AI enabled)
ai_voting_enabled = true   # AI scores influence validator selection
ai_weight = 0.3            # Weight of AI score vs stake
ai_threshold_valid = 0.8   # Minimum AI confidence for validator
```

**Example:** PoES mode (default):
```toml
[consensus]
consensus = "poes"

[consensus.poes]
enabled = true
security_constant = 1.2
min_security_level = 0.99
```

### HF (Hard Fork) Activation

```toml
[consensus.hf]
# Define activation heights or timestamps
hf1_height = 100000       # Activate at block height
hf2_timestamp = 1704067200 # Activate at Unix timestamp

# Deployment method
# "height" - activate at fixed block
# "timestamp" - activate at fixed time
# "miner" - 75% of miners signal via coinbase
deployment = "height"

# Signal bits (miner voting)
signal_bits = 0x20000000  # Bit 29
signaling_period = 2016   # Blocks in voting period
required_ratio = 0.95     # 95% of blocks must signal
```

---

## Cryptography

### Hash Functions

```toml
[crypto]
# Use SHA-3 (Keccak) instead of SHA-256
sha3_enabled = true

# Quantum-resistant signatures
post_quantum_sigs = true    # Enable SPHINCS+ (slower, much larger)
# Fallback to ECDSA for compatibility
allow_fallback = false      # If true, allows ECDSA as fallback
```

### Key Management

```toml
[crypto.keys]
# Encryption algorithm for wallet/keys
encryption_algorithm = "aes-256-cbc"

# Key derivation (PBKDF2)
kdf_iterations = 100000    # Higher = more secure, slower unlock
kdf_memory = 256           # Memory-hard KDF (Argon2 optional)

# Mnemonic (BIP-39)
mnemonic_language = "english"  # "english", "chinese", "japanese"
mnemonic_strength = 128         # 128/160/192/224/256 bits (12/15/18/21/24 words)

# Address format
address_version = 76       # P2PKH version byte (0x4C for NFX mainnet/testnet)
script_version = 18        # P2SH version byte
bech32_hrp = "nfx"        # Human-readable prefix for bech32 addresses
```

---

## AI Governance

### AI Engine Settings

```toml
[ai]
enabled = true              # Enable AI validation (highly recommended)

# Model configuration
model_path = "/opt/nfx/models/ai_v2.bin"  # Pre-trained neural network weights
model_format = "tiny-dnn"                 # Framework: "tiny-dnn", "onnx", "tflite"

# Confidence thresholds (0.0 to 1.0)
threshold_valid = 0.80      # Block accepted if score > 0.80
threshold_suspicious = 0.60 # Block flagged if score in [0.6, 0.8)
threshold_malicious = 0.40  # Block rejected if score < 0.40 (rare)

# Action thresholds
auto_ban = false           # Automatically ban peers submitting malicious blocks
auto_flag = true           # Flag suspicious blocks for human review

# Scanning
scan_interval = "1m"       # How often to scan recent blocks ("30s", "5m", "1h")
scan_batch_size = 100      # Blocks per scan batch

# Features extracted from blocks
features = [
    "tx_count",
    "tx_size_avg",
    "tx_size_stddev",
    "fee_median",
    "fee_percentile_25",
    "fee_percentile_75",
    "scriptsig_ratio",     # % of non-standard scripts
    "time_variance",       # Median time - min time
    "duplicate_txids",     # Count of duplicate TXIDs
    "coinbase_rate",       # Coinbase size / block size
    "op_return_count",     # OP_RETURN outputs
    "stake_operations"     # Stake-related ops
]

# Minimum peers required before AI scanning starts
min_peers_for_ai = 10
```

### AI Retraining (Administrator Only)

```bash
# Train AI model offline (requires labeled dataset)
nfx-ai-train \
    --dataset labeled_blocks.jsonl \
    --epochs 200 \
    --output /opt/nfx/models/ai_v2.bin

# Test model accuracy
nfx-ai-test --model /opt/nfx/models/ai_v2.bin --testset test_data.jsonl
```

---

## Database (LevelDB)

### LevelDB Options

```toml
[database]
# Cache size in MB (per shard, if sharded)
dbcache = 256

# Maximum mempool size (MB)
maxmempool = 128

# How long to keep transactions in mempool (hours)
mempool_maxtime = 72

# Database compaction strategy
compaction = "auto"       # "auto", "manual", "aggressive", "disabled"

# LevelDB specific (advanced)
compression = "snappy"    # "none", "snappy", "lz4", "zstd"
write_buffer_size = 64    # MB per memtable
max_open_files = 1000     # Max file descriptors for DB

# Fast shutdown (skip flushing cache)
# false = slower startup but safer; true = faster but risk of data loss on power failure
write_ahead_log = true

# Block pruning (archive node only)
prune = false             # true = keep only UTXO set, discard old blocks
prune_target_mb = 55000   # Target DB size for pruning (55GB)
```

### Database Paths

```toml
# Chainstate (UTXO set, block indexes)
chainstate = "chainstate"  # Subdirectory of datadir

# Blocks (raw block data)
blocks = "blocks"

# Indexes (txindex=1)
indexes = "indexes"

# Peers database
peers = "peers"

# Wallet database (if wallet RPC enabled)
walletdir = "wallets"
```

---

## Virtual Machine (Smart Contracts)

### JavaScript VM (Duktape)

```toml
[vm]
enabled = true               # Enable VM execution

# Gas system
gas_limit = 2100000         # Max gas per transaction
gas_price = 1               # Minimum gas price (satoshis per gas)
minimum_gas_price = 1       # Enforced minimum
max_gas_per_block = 10000000  # Block gas limit (~1500 tps)

# Execution limits (anti-DoS)
max_script_operations = 100000  # Max opcodes per script
script_timeout = 5              # Execution timeout (seconds)
max_stack_items = 1000          # VM stack size limit
max_call_depth = 32             # Reentrancy protection

# Features
enable_wasm = false          # Experimental WebAssembly support
enable_events = true         # Emit events for contract state changes

# Pre-compiled contracts (native functions)
precompiles = [
    "0x01:keccak256",
    "0x02:sha256",
    "0x03:ripemd160",
    "0x04:identity"
]
```

### Gas Schedule

| Operation | Gas Cost |
|-----------|----------|
| `ADD` | 3 |
| `MUL` | 5 |
| `SSTORE` (set from 0â†’non-zero) | 20000 |
| `SSTORE` (delete) | 5000 |
| `SUICIDE` | 5000 |
| `BALANCE` | 100 |
| `SLOAD` | 200 |
| `SELFDESTRUCT` | 5000 |
| `LOG` | 375 + 375 Ã— topics |
| `CALL` | 700 |
| `CREATE` | 32000 |

### Contract Deployment

```toml
[vm.deploy]
# Maximum contract bytecode size (bytes)
max_code_size = 24576  # 24 KB

# Minimum required gas for deployment
min_gas_deploy = 1000000

# Enable CREATE2 (deterministic addresses)
enable_create2 = true
```

---

## RPC Server

### Authentication

```toml
[rpc]
# HTTP Basic Auth
rpcuser = "nfxuser"
rpcpassword = "CHANGE_THIS_TO_STRONG_RANDOM_PASSWORD"

# Whitelist (default: localhost only)
rpcallowip = "127.0.0.1"

# Allow multiple IPs
# rpcallowip = ["127.0.0.1", "192.168.1.0/24"]

# Maximum concurrent HTTP connections
maxconnections = 128

# Request timeout (seconds)
rpc_timeout = 30

# Enable CORS (for web apps)
# cors = "http://localhost:3000"

# TLS/HTTPS (recommended for remote access)
# Use reverse proxy (nginx) for TLS termination
# Built-in TLS is experimental:
# rpc_tls_cert = "/etc/ssl/certs/nfx.crt"
# rpc_tls_key = "/etc/ssl/private/nfx.key"
```

### RPC Methods Whitelist

Restrict available methods:

```toml
[rpc.whitelist]
# Default: all methods enabled
# Format: "method1", "method2", ...

# Public read-only methods
public = [
    "getblockchaininfo",
    "getblockcount",
    "getblock",
    "gettransaction",
    "getbalance",
    "listunspent"
]

# Write methods (require authentication)
private = [
    "sendtransaction",
    "walletcreate",
    "walletpassphrase"
]

# Admin-only
admin = [
    "setban",
    "disconnectnode",
    "getmemoryinfo"
]
```

### WebSocket

```toml
[websocket]
enabled = true
bind = "127.0.0.1:18334"

# Notifications
notifications = [
    "block",           # New block
    "tx",              # Transaction inclusion
    "rawtx",           # Mempool transaction
    "alert",           # Network alerts
    "ai_score"         # AI governance results
]

# Rate limiting (messages/sec per connection)
rate_limit = 100
```

---

## Wallet

### Wallet Settings

```toml
[wallet]
enabled = true               # Enable wallet RPC methods
walletdir = "wallets"        # Subdir of datadir
default_wallet = ""          # Auto-load this wallet on startup (UUID)

# Encryption
encrypt_wallet = true        # Encrypt wallet on disk
auto_encrypt = false         # Auto-encrypt after creation

# Backups
auto_backup = true           # Create encrypted backups
backup_interval = "7d"       # "1d", "1w", "1M"
backup_retention = 30        # Keep N backups

# Address generation
gap_limit = 20               # Generate N addresses ahead
change_gap_limit = 10        # Change addresses gap

# Display
balance_precision = 8        # Decimal places for NFX (typically 8)
fiat_currency = "USD"        # "USD", "BTC", "EUR", etc.
show_zero_confirmations = false  # Hide 0-conf transactions
```

### Staking (PoS/PoES)

```toml
[staking]
enabled = true

# Minimum stake amount (NFX, assuming 8 decimals)
min_stake = 10000000000    # 100 NFX

# Staking lockup period (blocks)
lockup_blocks = 1440      # 24 hours (60s blocktime Ã— 1440 = 1 day)

# Rewards
reward_percent = 0.05     # 5% annual inflation distributed to stakers
payout_interval = 1440    # Reward payout every 24h

# Maximum staked percentage of total supply
max_stake_percent = 0.60  # 60% max staked

# Auto-staking
auto_stake = false        # Automatically stake received funds
```

---

## Performance

### CPU & Threads

```toml
[performance]
# Number of worker threads (0 = auto-detect CPU cores)
threads = 0

# Script verification parallelism
max_parallel_scripts = 4

# PMEM (Persistent Memory) support
use_pmem = false
pmem_path = "/dev/dax0.0"  # Intel Optane DC

# Preload chain into RAM on startup
preload = false

# Reduce memory usage at cost of performance
low_memory = false
```

### Networking

```toml
[network.performance]
# Buffers
recvbuffer = 4096          # Socket receive buffer (KB)
sendbuffer = 4096          # Socket send buffer (KB)

# Connection limits
maxuploadtarget = 5000     # KB/sec upload limit (0 = unlimited)
maxsendbuffer = 1024       # MB per connection send buffer

# Fast propagation
compact_blocks = true      # Use compact block relay
compact_block_ratio = 0.85  # Transactions to skip in relay
```

### Disk I/O

```toml
[io]
# Flush mode
# "periodic" = flush every n seconds (safer)
# "commit" = fsync after every block (slower, safest)
# "off" = rely on OS (fastest, riskier)
flush_mode = "periodic"

flush_interval = 600       # Flush every 10 minutes (if periodic)

# Write-ahead log (WAL) mode for LevelDB
wal_enabled = true
wal_size_mb = 16
wal_autoflush = true

# Async I/O (Linux io_uring)
use_io_uring = false      # Experimental, requires kernel 5.1+
```

---

## Logging

### Structured Logging

```toml
[log]
# Format: "text" or "json"
format = "text"

# Log levels: trace, debug, info, warn, error, fatal
level = "info"

# Rotate logs
rotate_size_mb = 100
rotate_count = 7

# Include fields (json format only)
include_fields = [
    "timestamp",
    "level",
    "logger",
    "message",
    "module",
    "thread"
]
```

### Log Categories

Set minimum level per category:

```toml
[log.categories]
net = "info"           # P2P networking
consensus = "info"     # Block validation, mining
rpc = "warn"           # RPC requests (reduce noise)
db = "warn"            # Database operations
vm = "debug"           # Smart contract execution
ai = "info"            # AI governance
wallet = "info"        # Wallet operations
```

### External Log Aggregation

```toml
[log.remote]
enabled = false
type = "loki"           # "loki", "splunk", "elasticsearch"
endpoint = "http://localhost:3100/loki/api/v1/push"
batch_size = 100
batch_timeout = "5s"
```

---

## Examples

### 1. Minimal Testnet Config

```toml
# ~/.nfx/config.toml
[network]
testnet = true
rpc_port = 18332
p2p_port = 18333

[rpc]
rpcuser = "test"
rpcpassword = "test123"
rpcallowip = "127.0.0.1"

[consensus]
consensus = "poes"
blocktime = 60

[database]
dbcache = 128
maxmempool = 64

[logging]
console = true
logcategories = ["net", "rpc"]
```

### 2. Production Mainnet Node

```toml
# /etc/nfx/config.toml
[network]
testnet = false
p2p_port = 8333
rpc_port = 8332
rpc_bind = "127.0.0.1:8332"
maxconnections = 125

dns_seeds = [
    "seed.mainnet.nfxchain.io",
    "seed2.mainnet.nfxchain.io"
]

[rpc]
rpcuser = "admin"
rpcpassword = "RANDOM_64_CHAR_PASSWORD_HERE"
rpcallowip = ["127.0.0.1", "10.0.0.0/8"]

[consensus]
consensus = "poes"
blocktime = 60

[ai]
enabled = true
threshold_valid = 0.85

[database]
dbcache = 2048
maxmempool = 256
compaction = "auto"

[wallet]
enabled = true

[performance]
threads = 8
preload = true

[logging]
console = false
logfile = "/var/log/nfx/nfx.log"
maxlogfilesize = 100
maxlogfiles = 30
logcategories = ["net", "consensus", "rpc", "wallet"]

[log.categories]
rpc = "warn"
```

### 3. Hypernode (Guardian) Config

```toml
[hypernode]
enabled = true

# Exclude peers (privacy)
exclude_peers = ["badnode1", "badnode2"]

# Maximum concurrent AI validations
max_ai_parallel = 4

# Bonded stake info
bonded_amount = 100000000000  # 1000 NFX (example)
bond_txid = "abc123..."

# Payout address for rewards
payout_address = "NFX1payout..."

[performance]
threads = 16
dbcache = 4096
preload = true

[ai]
enabled = true
threshold_valid = 0.90  # Stricter for hypernodes
```

### 4. Disabled Features (Minimal Node)

```toml
[wallet]
enabled = false

[ai]
enabled = false

[vm]
enabled = false

[staking]
enabled = false

# Result: Lightweight block-relay only node
```

---

## Command-Line Flags

Configuration can also be set via flags:

```bash
nfx-node \
    --testnet \
    --rpcuser=alice \
    --rpcpassword=secret \
    --datadir=/mnt/ssd/nfx \
    --dbcache=2048 \
    --maxconnections=125 \
    --log-level=info
```

**Flag â‡„ Config mapping:**

| Flag | Config | Type |
|------|--------|------|
| `--testnet` | `[network] testnet = true` | Boolean |
| `--mainnet` | `[network] testnet = false` | Boolean |
| `--datadir PATH` | `datadir` | String |
| `--dbcache N` | `[database] dbcache` | Integer (MB) |
| `--maxconnections N` | `[network] maxconnections` | Integer |
| `--rpcuser USER` | `[rpc] rpcuser` | String |
| `--rpcpassword PASS` | `[rpc] rpcpassword` | String |
| `--pidfile PATH` | `pidfile` | String |
| `--logfile PATH` | `logfile` | String |
| `--log-level LEVEL` | `[log] level` | String (debug/info/warn/error) |
| `--daemon` | N/A | Boolean (run in background) |
| `--console` | N/A | Boolean (log to console) |
| `--reindex` | N/A | Boolean (reindex chainstate) |

---

## Environment Variables

Use environment variables (prefixed `NFX_`) for secrets or CI/CD:

```bash
export NFX_DATADIR="/var/lib/nfx/data"
export NFX_RPCUSER="nfxuser"
export NFX_RPCPASSWORD="supersecret"
export NFX_TESTNET="1"  # '1' = true, '0' = false

nfx-node  # Reads env vars automatically
```

**Supported variables:**

| Variable | Config |
|----------|--------|
| `NFX_DATADIR` | `datadir` |
| `NFX_LOGFILE` | `logfile` |
| `NFX_LOGCATEGORIES` | `[log] categories` |
| `NFX_LOGLEVEL` | `[log] level` |
| `NFX_RPCUSER` | `[rpc] rpcuser` |
| `NFX_RPCPASSWORD` | `[rpc] rpcpassword` |
| `NFX_TESTNET` | `[network] testnet` |
| `NFX_MAINNET` | `[network] testnet = false` |
| `NFX_DBCACHE` | `[database] dbcache` |

---

*Next: [FAQ](../faq.md) | [Architecture](../architecture.md)*
