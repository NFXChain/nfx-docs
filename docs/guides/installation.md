# Installation & Building Guide

This guide covers building NFX Chain from source on Linux (Ubuntu/Debian). macOS and Windows (WSL2) are also supported with minor variations.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Ubuntu/Debian Installation](#ubuntudebian-installation)
3. [macOS Installation](#macos-installation)
4. [Building All Modules](#building-all-modules)
5. [Automated Build Script](#automated-build-script)
6. [Configuration](#configuration)
7. [Running a Node](#running-a-node)
8. [Verifying Installation](#verifying-installation)
9. [Troubleshooting](#troubleshooting)

## System Requirements

### Minimum Requirements
- **CPU**: 2 cores (4+ recommended)
- **RAM**: 4 GB (8+ GB for running node + wallet)
- **Disk**: 50 GB free (SSD recommended)
- **OS**: Ubuntu 22.04 LTS, Debian 12, or compatible

### Recommended Development Setup
- **CPU**: 8+ cores
- **RAM**: 16 GB
- **Disk**: 200 GB NVMe SSD
- **OS**: Ubuntu 22.04 LTS

### Software Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| CMake | â‰¥ 3.10 | Build system |
| GCC/Clang | â‰¥ 9.0 / â‰¥ 10.0 | C++ compiler (C++17) |
| Qt5 | â‰¥ 5.12 | GUI framework & core libraries |
| LevelDB | â‰¥ 1.22 | Key-value storage |
| OpenSSL | â‰¥ 1.1.1 | Cryptography library |
| Boost | â‰¥ 1.74 | Utility libraries |
| Go | â‰¥ 1.19 | API & CLI tools |

## Ubuntu/Debian Installation

### Step 1: Install System Dependencies

```bash
# Update package list
sudo apt update

# Install build essentials
sudo apt install -y \
    build-essential \
    cmake \
    git \
    curl \
    wget \
    pkg-config

# Install Qt5
sudo apt install -y \
    qtbase5-dev \
    qt5-qmake \
    qtbase5-dev-tools

# Install LevelDB
sudo apt install -y libleveldb-dev

# Install additional libraries
sudo apt install -y \
    libssl-dev \
    libboost-all-dev \
    libjsoncpp-dev

# Verify installations
gcc --version     # Should be â‰¥ 9.0
cmake --version   # Should be â‰¥ 3.10
qmake --version   # Should show Qt 5.x
```

### Step 2: Install Go (for nfx-go module)

```bash
# Download Go 1.19+
wget https://go.dev/dl/go1.19.13.linux-amd64.tar.gz

# Remove old installation (if exists)
sudo rm -rf /usr/local/go

# Extract to /usr/local
sudo tar -C /usr/local -xzf go1.19.13.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc

# Reload shell configuration
source ~/.bashrc

# Verify installation
go version  # Should print: go version go1.19.13 linux/amd64
```

### Step 3: Clone Repository

```bash
# Clone the main repository
git clone https://github.com/NFXChain/nfx-chain.git
cd nfx-chain

# View structure
tree -L 2 -d
# .
# â”œâ”€â”€ nfx-core/
# â”œâ”€â”€ nfx-bindings/
# â”œâ”€â”€ nfx-go/
# â”œâ”€â”€ nfx-wallet/
# â”œâ”€â”€ build_nfx.sh
# â””â”€â”€ README.md
```

## Building All Modules

### Build nfx-core

```bash
cd nfx-core

# Create build directory
mkdir build && cd build

# Configure with CMake (Release for production)
cmake .. -DCMAKE_BUILD_TYPE=Release

# Alternative configurations:
# cmake .. -DCMAKE_BUILD_TYPE=Debug    # For debugging
# cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local  # System install

# Compile using all CPU cores
make -j$(nproc)

# Or specify number of jobs (e.g., 4 cores)
# make -j4

# Estimated build time: 5-15 minutes depending on CPU

# Verify build
ls -lh nfx-node
# -rwxr-xr-x 2.1M nfx-node

# Run unit tests (optional)
make test

# Install to system (optional)
# sudo make install
# sudo ldconfig  # Refresh shared library cache
```

**Build artifacts:**
- `nfx-node` - Main node executable
- `libnfx-core.so` - Shared library (if built as shared)
- `quantum_crypto.a` - Static crypto library

### Build nfx-bindings

```bash
cd ../nfx-bindings

# Create build directory
mkdir build && cd build

# Configure
cmake ..

# Build
make -j$(nproc)

# Output: libnfx.so (shared library for Go/Python/other languages)
ls -lh libnfx.so
```

**Note:** The bindings module automatically links against `nfx-core`. Ensure `nfx-core` is built first or provide the library path:

```bash
# If nfx-core not installed system-wide:
cmake .. \
    -DNFX_CORE_INCLUDE_DIR=../../core/include \
    -DNFX_CORE_LIBRARY=../../core/build/libnfx-core.so
```

### Build nfx-go

```bash
cd ../nfx-go

# Ensure Go environment is set
go env

# Download dependencies
go mod download

# Build the node
go build -o nfx-node ./api

# Or build with optimizations
go build -ldflags="-s -w" -o nfx-node ./api

# Build CLI tool
go build -o nfx-cli ./cli

# Verify
./nfx-node --version
./nfx-cli --help
```

**Go module dependencies** (automatically handled by `go mod`):
- `github.com/gin-gonic/gin` - HTTP framework
- `github.com/gorilla/websockets` - WebSocket support
- `golang.org/x/crypto` - Cryptographic primitives

### Build nfx-wallet (Optional)

```bash
cd ../nfx-wallet

# Ensure Qt5 is properly configured
export Qt5_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5
export PATH="/usr/lib/qt5/bin:$PATH"

# Create build directory
mkdir build && cd build

# Configure with CMake
cmake .. -DCMAKE_BUILD_TYPE=Release

# If Qt not found automatically, specify:
# cmake .. -DCMAKE_PREFIX_PATH=/usr/lib/x86_64-linux-gnu/qt5

# Build (requires significant RAM ~8GB)
make -j$(nproc)

# Output: nfx-wallet binary
ls -lh nfx-wallet

# Run wallet
./nfx-wallet
```

**Note:** Building the wallet requires Qt5 development packages. If you encounter errors, install:

```bash
sudo apt install qtbase5-dev qt5-qmake qtbase5-dev-tools \
                 qtdeclarative5-dev qml-module-qtquick2 \
                 qml-module-qtquick-window2 qml-module-qtquick-dialogs
```

## Automated Build Script

The repository includes `build_nfx.sh` for automated setup on Ubuntu:

```bash
# From project root
chmod +x build_nfx.sh
./build_nfx.sh --testnet

# Options:
#   --testnet       Use testnet (default)
#   --mainnet       Use mainnet configuration
#   --no-wallet     Skip wallet compilation
#   --skip-deps     Skip dependency installation (if already installed)
#   --workspace DIR Set custom workspace directory
```

**What the script does:**
1. Checks OS (Ubuntu/Debian only)
2. Installs all dependencies via apt
3. Sets up workspace in `~/nfx-workspace`
4. Clones all submodules (if needed)
5. Builds nfx-core, nfx-bindings, nfx-go, and nfx-wallet
6. Creates node directory at `~/nfx-testnet` or `~/nfx-mainnet`
7. Generates default configuration
8. Creates startup scripts

**Manual vs Automated:**
- Manual build gives you control over each module
- Automated script is ideal for quick testnet deployment
- Both produce identical binaries

## Configuration

### config.toml Structure

After building, create a configuration file:

```bash
# Copy example config
cp nfx-go/config.example.toml nfx-go/config.toml
nano nfx-go/config.toml
```

**Complete configuration reference:**

```toml
# â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
# â”‚ NFX Chain Node Configuration                             â”‚
# â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

# â”€â”€ General â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
datadir = "/home/user/.nfx/data"         # Data directory
logfile = "/home/user/.nfx/logs/nfx.log" # Log file path
loglevel = "info"                        # debug|info|warn|error
maxlogfilesize = 50                      # MB per log file
maxlogfiles = 10                         # Keep N rotated logs

# â”€â”€ Network â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[network]
testnet = true             # true=testnet, false=mainnet
p2p_port = 18333          # Port for P2P connections (default: 18333 testnet, 8333 mainnet)
rpc_port = 18332          # Port for RPC API (default: 18332 testnet, 8332 mainnet)
rpc_bind_address = "127.0.0.1"  # Bind address for RPC
maxconnections = 50       # Maximum peer connections
maxoutbound = 8           # Max outbound connections
max inbound = 42          # Max inbound connections
dns_seeds = [
    "seed.testnet.nfxchain.io",
    "seed2.testnet.nfxchain.io"
]
banned_nodes = []         # List of banned peer IDs

# â”€â”€ Consensus â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[consensus]
consensus = "poes"        # pow|pos|poes
powdifficulty = 1         # PoW difficulty target (bits)
powallowminertips = true  # Allow tips to miner
posminstake = 1000        # Minimum PoS stake in NFX
posesecurityfactor = 1.0  # PoES security multiplier (dynamic)
blocktime = 60            # Target block time in seconds
maxt blocksize = 2000000  # 2MB max block size
mintxfee = 1000           # Minimum transaction fee (satoshis)

# â”€â”€ Cryptography â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[crypto]
sha3_enabled = true       # Use SHA-3 instead of SHA-256
post_quantum_sigs = true  # Enable SPHINCS+ signatures
cert_cache_size = 1000    # Certificate cache size

# â”€â”€ AI Governance â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[ai]
enabled = true           # Enable AI validation
model_path = "/home/user/.nfx/models/"  # Neural network weights
threshold_valid = 0.8    # Confidence threshold for "valid"
threshold_suspicious = 0.6  # Threshold for "suspicious"
auto_ban = false         # Automatically ban malicious nodes
min_peers_for_ai = 10    # Minimum peers before AI runs

# â”€â”€ Database (LevelDB) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[database]
dbcache = 256            # Database cache in MB
maxmempool = 128         # Mempool max size in MB
mempool_maxtime = 72     # Hours to keep tx in mempool
compaction = "auto"      # auto|manual|aggressive

# â”€â”€ Virtual Machine (JavaScript) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[vm]
enabled = true           # Enable smart contract VM
max_gas_per_tx = 2100000 # Gas limit per transaction
gas_price = 1            # Minimum gas price
enable_wasm = false      # Experimental: WebAssembly support
vm_timeout = 5           # Max execution time in seconds

# â”€â”€ RPC Server â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[rpc]
rpcuser = "nfxuser"      # RPC username
rpcpassword = "change_this!"  # RPC password (use strong password!)
rpcallowip = "127.0.0.1" # Allowed IP (use firewall)
rpc_port = 8332          # Mainnet port (change for testnet)
rpc_timeout = 30         # Request timeout in seconds
max_requests = 100       # Max concurrent requests

# â”€â”€ Wallet â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[wallet]
enabled = true           # Enable wallet RPC methods
walletdir = "/home/user/.nfx/wallets"  # Wallet storage
default_wallet = ""      # Auto-load this wallet
fee_priority = 1         # Fee priority (1=low, 3=high)

# â”€â”€ Performance â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[performance]
threads = 0               # 0 = auto-detect CPU cores
max_parallel_scripts = 4  # Parallel VM execution threads
preload = false           # Preload chain into memory on startup

# â”€â”€ Logging â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
[logging]
console = true           # Log to stdout
logtimestamps = true     # Include timestamps
logthreadnames = false   # Include thread names
logcategories = ["net", "consensus", "rpc"]  # Filter by category
```

### Security Notes

**âš ï¸ IMPORTANT:**
- Change `rpcpassword` to a strong, unique value
- Never expose RPC port to public internet (use SSH tunnel or VPN)
- Use firewall rules: `sudo ufw allow from 192.168.1.0/24 to any port 8332`
- For mainnet, use `rpcallowip = "127.0.0.1"` only

## Running a Node

### 1. Using nfx-go Binary Directly

```bash
# Navigate to nfx-go directory
cd nfx-go

# Basic run (uses default config.toml if exists)
./nfx-node --config config.toml

# With explicit config
./nfx-node --config /path/to/config.toml

# With custom data directory
./nfx-node --datadir /home/user/.nfx/testnet

# Run as daemon (background)
nohup ./nfx-node --config config.toml > nfx.log 2>&1 &

# Check if running
ps aux | grep nfx-node

# View logs
tail -f nfx.log
```

### 2. Using systemd Service (Production)

Create a systemd unit file:

```bash
sudo nano /etc/systemd/system/nfx-node.service
```

```ini
[Unit]
Description=NFX Chain Node
After=network.target

[Service]
Type=simple
User=yourusername
Group=yourusername
WorkingDirectory=/home/yourusername/nfx-go
ExecStart=/home/yourusername/nfx-go/nfx-node --config /home/yourusername/.nfx/config.toml
Restart=always
RestartSec=10
LimitNOFILE=65535

# Security hardening
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
NoNewPrivileges=true

# Resource limits
MemoryLimit=4G
CPUQuota=300%

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable nfx-node
sudo systemctl start nfx-node

# Check status
sudo systemctl status nfx-node

# View logs
sudo journalctl -u nfx-node -f
```

### 3. Using Docker (Alternative)

```dockerfile
# Dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    nfx-core \
    nfx-go \
    && rm -rf /var/lib/apt/lists/*

COPY config.toml /etc/nfx/config.toml

EXPOSE 18332 18333

CMD ["/usr/bin/nfx-node", "--config", "/etc/nfx/config.toml"]
```

Build and run:

```bash
docker build -t nfx-node .
docker run -d \
    -v /home/user/.nfx/data:/data \
    -p 18332:18332 -p 18333:18333 \
    --name nfx-node \
    nfx-node
```

## Verifying Installation

### Check Node Version

```bash
cd nfx-go
./nfx-node --version
# Expected output: NFX Node version 1.0.0 (build: xxxx)
```

### Check RPC Connectivity

```bash
# Using nfx-cli
./nfx-cli getinfo

# Or using curl
curl --user nfxuser:yourpassword \
     --data-binary '{"jsonrpc":"1.0","id":"curltest","method":"getblockchaininfo","params":[]}' \
     -H 'content-type: text/plain;' \
     http://127.0.0.1:18332/
```

Expected response:
```json
{
  "result": {
    "chain": "testnet",
    "blocks": 0,
    "headers": 0,
    "bestblockhash": "...",
    "difficulty": 1,
    "verificationprogress": 0.0
  },
  "error": null,
  "id": "curltest"
}
```

### Run Tests

```bash
# nfx-core unit tests
cd nfx-core/build
ctest   # or: ctest -VV for verbose

# Go API tests
cd ../nfx-go
go test ./...

# Benchmark (optional)
go test -bench=. ./...
```

### Check Synchronization

```bash
# Get current block count
./nfx-cli getblockcount

# Or via RPC
curl --user nfxuser:password \
     -d '{"jsonrpc":"1.0","id":"test","method":"getblockcount","params":[]}' \
     http://127.0.0.1:18332/
```

## Troubleshooting

### CMake Cannot Find Qt5

**Error:** `Could NOT find Qt5 (missing: Qt5Core_DIR)`

**Solution:**
```bash
# Find Qt5 location
find /usr -name "Qt5Config.cmake" 2>/dev/null

# Typically: /usr/lib/x86_64-linux-gnu/cmake/Qt5

# Set Qt5_DIR environment variable
export Qt5_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5

# Or pass to cmake directly
cmake .. -DQt5_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5
```

### LevelDB Not Found

**Error:** `Could NOT find LevelDB (missing: LEVELDB_INCLUDE_DIR LEVELDB_LIBRARY)`

**Solution:**
```bash
# Install development package
sudo apt install libleveldb-dev

# Or build from source if version too old:
git clone https://github.com/google/leveldb.git
cd leveldb
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install
sudo ldconfig
```

### Go Build Fails: "cannot find module"

**Error:** `go: module requires Go 1.20`

**Solution:**
```bash
# Update Go to latest version
go version  # Check current
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

# Update PATH
source ~/.bashrc

# Verify
go version  # Should show 1.21.5
```

### Out of Memory During Wallet Build

**Error:** `g++: internal compiler error: Killed (program cc1plus)`

**Solution:**
```bash
# Reduce parallel jobs
make -j2   # Use only 2 cores instead of all

# Or increase RAM (swap)
sudo swapoff -a
sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
sudo swapon /swapfile

# After build, you can remove swap
sudo swapoff /swapfile
sudo rm /swapfile
```

### Node Fails to Start: "Address already in use"

**Error:** `bind: address already in use`

**Solution:**
```bash
# Check what's using the port
sudo lsof -i :8332  # RPC port
sudo lsof -i :8333  # P2P port

# Kill conflicting process
sudo kill $(sudo lsof -t -i:8332)

# Or change ports in config.toml
rpc_port = 18332  # Use testnet port
p2p_port = 18333
```

### "Assertion failed" on Startup

Usually indicates database corruption or incompatible version:

```bash
# Backup and reset datadir
mv ~/.nfx ~/.nfx.backup
mkdir -p ~/.nfx/data

# Re-sync blockchain from scratch
./nfx-node --datadir ~/.nfx
```

### Slow Synchronization

If sync is very slow:

```toml
# In config.toml, increase connections:
[network]
maxconnections = 100

# Increase database cache:
[database]
dbcache = 1024  # Use 1GB cache (if you have RAM)
```

### Wallet Won't Connect to Node

1. Ensure node RPC is running and accessible:
   ```bash
   curl http://127.0.0.1:18332/
   ```

2. Check RPC credentials in wallet config match `config.toml`

3. Verify firewall allows localhost connections

### Build Succeeds But Binary Crashes

```bash
# Run with debug logging
./nfx-node --log-level=debug --datadir=/tmp/test 2>&1 | tee debug.log

# Check for missing shared libraries
ldd nfx-node | grep "not found"

# If missing, install or set LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

### Additional Resources

- **GitHub Issues**: [Report bugs](https://github.com/NFXChain/nfx-chain/issues)
- **Discord**: [Community support](https://discord.gg/nfx)
- **Wiki**: [Detailed guides](https://github.com/NFXChain/nfx-chain/wiki)

---

*Next: [Module Documentation](../modules/) | [API Reference](../api/) | [Examples](../examples/)*
