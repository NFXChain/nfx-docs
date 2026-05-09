# Frequently Asked Questions

## General Questions

### **Q: What is NFX Chain?**

NFX Chain is a next-generation blockchain platform combining quantum-resistant cryptography, AI-driven governance, and a hybrid consensus mechanism (PoW/PoS/PoES) for enhanced security and scalability.

### **Q: What does PoES mean?**

PoES (Proof of Exponential Security) is NFX Chain's novel consensus algorithm. Security increases exponentially as the network grows, making 51% attacks economically infeasible on large networks.

### **Q: Is NFX Chain really quantum-resistant?**

Yes. NFX uses SHA-3 (Keccak) for hashing and SPHINCS+ for digital signatures, both considered secure against known quantum attacks (Shor's algorithm, Grover's algorithm).

### **Q: What's the total supply of NFX?**

Total supply: **100,000,000 NFX** (with 8 decimal places, so 10^16 satoshis). Fixed cap, no inflation after block 1,000,000 (PoES phase).

---

## Installation & Building

### **Q: I get "CMake not found" error. How to fix?**

Install CMake:

```bash
# Ubuntu/Debian
sudo apt install cmake

# Verify
cmake --version  # Should be â‰¥ 3.10
```

### **Q: Qt5 not found during compilation**

```bash
# Install Qt5 development packages
sudo apt install qtbase5-dev qt5-qmake qtbase5-dev-tools

# Find Qt5 path
find /usr -name "Qt5Config.cmake" 2>/dev/null

# Set Qt5_DIR in CMake
cmake .. -DQt5_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5
```

### **Q: "fatal error: LevelDB/leveldb/db.h: No such file"**

Install LevelDB dev package:

```bash
sudo apt install libleveldb-dev
```

Or build from source:

```bash
git clone https://github.com/google/leveldb.git
cd leveldb && mkdir build && cd build
cmake .. && make -j$(nproc) && sudo make install
sudo ldconfig
```

### **Q: Out of memory while compiling wallet**

The Qt wallet requires ~8GB RAM to compile. Use swap or reduce parallel jobs:

```bash
# Reduce to 2 parallel jobs
make -j2

# Or create swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Remove swap after build
sudo swapoff /swapfile
sudo rm /swapfile
```

---

## Running a Node

### **Q: How do I start the node?**

```bash
# Quick testnet start
nfx-node --testnet

# With config file
nfx-node --config ~/.nfx/config.toml

# As daemon (background)
nfx-node --daemon --config config.toml

# Check status
nfx-cli getinfo
```

### **Q: Node won't start: "Address already in use"**

Port conflict (default RPC 18332 or P2P 18333):

```bash
# Check what's using port
sudo lsof -i :18332
sudo lsof -i :18333

# Kill or change ports in config.toml
```

### **Q: How long does initial sync take?**

First sync (from genesis): **6-48 hours** depending on:
- CPU speed (block validation is CPU-bound)
- Disk I/O (LevelDB reads)
- Network bandwidth (~10-20 GB to download chain)
- Peer connectivity (more peers = faster)

After initial sync, daily sync (catch-up) takes < 5 minutes.

### **Q: Node stuck syncing at 99%**

Usually indicates stalled peer or corrupted block:

```bash
# Restart node
nfx-cli stop
nfx-node --reindex-chainstate  # Fast resync (not full re-download)

# If still stuck, check logs
tail -f ~/.nfx/logs/nfx.log
```

---

## Wallet & Transactions

### **Q: How do I get testnet NFX?**

Use the official testnet faucet:

```bash
# Via Discord (recommended)
# Join https://discord.gg/nfx
# Channel: #testnet-faucet
# Command: /faucet <your_address>

# Or via API
curl -X POST https://faucet.testnet.nfxchain.io/api/request \
    -H "Content-Type: application/json" \
    -d '{"address":"NFX1..."}'
```

### **Q: Transaction stuck in mempool**

Common reasons:
1. **Fee too low** â€” Increase fee and rebroadcast
2. **Double spend** â€” Already spent those inputs
3. **Nonce too low** â€” Wrong sequence number
4. **Invalid script** â€” Malformed transaction

```bash
# Check mempool
nfx-cli getrawmempool

# Bump fee (replace-by-fee not yet implemented)
# Must create new transaction with higher fee

# Abandon stuck transaction
nfx-cli abandontransaction <txid>
```

### **Q: Transaction never confirms**

1. Verify fee is reasonable (â‰¥ 0.001 NFX)
2. Check if node is synced (`nfx-cli getblockcount`)
3. Check peer count (`nfx-cli getpeerinfo`)
4. Wait â€” blocks are 60 seconds, 6 confirmations = 6 minutes

### **Q: How to import private key?**

```bash
# Import WIF (Wallet Import Format)
nfx-cli importprivkey "L1aW4aubDFB7yfras2S1mN3bqg9w7YjY72..."

# Rescan blockchain (takes time)
nfx-cli importprivkey "L1..." "" true
```

âš ï¸ **Never importprivkey on a node with RPC open to internet!** Do offline or localhost only.

---

## Staking & Guardians

### **Q: Minimum stake for PoS/PoES?**

- **PoS validator:** 1,000 NFX minimum
- **Guardian/hypernode:** 10,000 NFX minimum
- **No maximum** â€” more stake = higher selection probability

### **Q: How to become a guardian/hypernode?**

1. Acquire â‰¥ 10,000 NFX
2. Run a stable, high-uptime node (99.9%+)
3. Register via special transaction:

```bash
nfx-cli registerguardian \
  --address=NFX1stake_address \
  --amount=10000000000  # 10,000 NFX (8 decimals)
```

4. Wait for inclusion in next hypernode set (approx. 100 blocks)
5. Begin earning rewards

### **Q: What are staking rewards?**

- **PoS:** ~5% annual inflation, distributed per-block proportional to stake
- **PoES:** Additional security rewards for hypernode operators (~3-5%)
- Rewards auto-compound if `auto_stake=true`

---

## Security

### **Q: Is my wallet secure?**

Yes, if you:
- âœ… Use strong, unique RPC password (â‰¥ 32 random characters)
- âœ… Bind RPC to localhost only (`rpcallowip = "127.0.0.1"`)
- âœ… Encrypt wallet with BIP-38 (AES-256)
- âœ… Never share mnemonic/private keys
- âœ… Keep system updated

### **Q: Should I expose RPC to internet?**

**NO.** RPC has no rate limiting and full wallet access. Use SSH tunnel or VPN:

```bash
# SSH tunnel (secure)
ssh -L 18332:localhost:18332 user@your-server.com

# Then connect localhost:18332
nfx-cli --rpcport=18332 getinfo
```

### **Q: How to change RPC password?**

Edit `config.toml`:

```toml
[rpc]
rpcpassword = "new_strong_password_here"
```

Restart node: `sudo systemctl restart nfx-node`

If wallet encrypted, decrypt â†’ change password â†’ re-encrypt:

```bash
# Decrypt old wallet
nfx-cli walletpassphrase "oldpassword" 600

# Change RPC password in config
nano ~/.nfx/config.toml

# Restart node
nfx-cli stop
nfx-node

# Re-encrypt wallet
nfx-cli walletlock
```

### **Q: Suspected key compromise?**

1. Create new wallet immediately:
   ```bash
   nfx-cli walletcreate "New Secure Wallet"
   ```

2. Transfer all funds to new address(es)

3. Encrypt new wallet with strong passphrase:
   ```bash
   nfx-cli encryptwallet "new-strong-passphrase"
   ```

4. **DO NOT** reuse old keys

5. Consider investigating breach vector (malware? weak password?)

---

## Troubleshooting

### **Q: Node keeps crashing**

Check logs:

```bash
# Last 50 lines
tail -n 50 ~/.nfx/logs/nfx.log

# Follow live
tail -f ~/.nfx/logs/nfx.log | grep -i error

# Look for "Segmentation fault", "panic", "assert"
```

Common fixes:
- **Out of memory:** Reduce `dbcache` in config
- **Corrupted DB:** `nfx-node --reindex-chainstate`
- **Old version:** Update to latest release

### **Q: High memory usage (>80%)**

Expected for initial sync. After sync, should be:

| Node Type | Expected RAM |
|-----------|--------------|
| Full node | 2-4 GB |
| Hypernode | 4-8 GB |
| Wallet only | 500 MB |

Reduce memory:

```toml
[database]
dbcache = 128     # Reduce from 256MB to 128MB
```

### **Q: Disk space filling up**

Log rotation:

```bash
# Edit config
[logging]
maxlogfilesize = 100   # MB per file
maxlogfiles = 7        # Keep 7 files (700MB total)
```

Enable pruning (if not needed for archival):

```toml
[database]
prune = true
prune_target_mb = 55000  # Keep 55GB (enough for 2+ months)
```

### **Q: Can't connect to peers**

Port forwarding (if behind NAT):

```bash
# Router: Forward port 18333 TCP to your server

# Check firewall
sudo ufw allow 18333/tcp
sudo ufw allow 8333/tcp  # Mainnet
```

Check DNS seeds:

```bash
# Verify DNS resolution
nslookup seed.testnet.nfxchain.io

# Add fixed peer
echo "addnode=192.168.1.100:18333" >> ~/.nfx/config.toml
```

### **Q: RPC connection refused**

Ensure RPC enabled and bound correctly:

```toml
[rpc]
rpcuser = "nfxuser"
rpcpassword = "password"
rpcallowip = "127.0.0.1"  # Default: localhost only
```

Test:

```bash
curl -u nfxuser:password \
  -d '{"jsonrpc":"1.0","id":"test","method":"getblockcount","params":[]}' \
  http://127.0.0.1:18332/
```

### **Q: Low peer count (< 8)**

Causes:
- Port not open (firewall/NAT)
- `maxconnections` set too low
- ISP blocking P2P

Fix: `http://canyouseeme.org` to test port 18333 open.

### **Q: Recovery from corrupted wallet?**

If wallet.dat is corrupted but you have mnemonic:

```bash
# Remove corrupted wallet
rm -rf ~/.nfx/wallets/

# Create new from mnemonic
nfx-cli walletcreatefrommnemonic "your twelve word seed phrase..."
```

If no backup, **wallet.dat** may be recoverable with photo recovery tools.

---

## Development

### **Q: How to contribute?**

Generally:

1. Fork repository
2. Create feature branch
3. Write tests
4. Run `make test` or `go test ./...`
5. Submit PR

### **Q: Running tests?**

```bash
# nfx-core C++ tests
cd nfx-core/build
ctest -VV  # Verbose

# nfx-go tests
cd nfx-go
go test ./... -v
go test -run TestGetBlock ./rpc

# nfx-wallet Qt tests
cd nfx-wallet/build
./test_wallet
```

### **Q: Debug logging**

```bash
# Specific module
nfx-node --log-categories="net,consensus" --log-level=debug

# All modules
nfx-node --log-level=trace

# To file only
nfx-node --log-file=/tmp/debug.log --log-level=debug --console=false
```

---

## Miscellaneous

### **Q: Windows support?**

Native Windows build **not officially supported**. Use:
- **WSL2** (Windows Subsystem for Linux)
- **Virtual Machine** (Ubuntu)
- **Docker Desktop** (Linux containers)

### **Q: Mobile wallet?**

Mobile wallets (iOS/Android) in development. Expected Q4 2026.

### **Q: Where to get help?**

- ðŸ“š **Docs:** https://docs.nfxchain.io
- ðŸ’¬ **Discord:** https://discord.gg/nfx
- ðŸ› **Issues:** https://github.com/NFXChain/nfx-chain/issues
- ðŸ“– **Wiki:** https://github.com/NFXChain/nfx-chain/wiki
- ðŸ“§ **Email:** support@nfxchain.io

### **Q: Reporting bugs**

Include:
1. OS and version (Ubuntu 22.04, macOS 13, etc.)
2. NFX version (`nfx-node --version`)
3. Log excerpts (last 100 lines)
4. Steps to reproduce
5. Expected vs actual behavior

```bash
# Collect logs for bug report
tar -czf nfx-logs-$(date).tar.gz ~/.nfx/logs/
```

---

*Still have questions? Ask on Discord or open a GitHub issue!*
