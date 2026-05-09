# Deployment & Operations Guide

Production deployment strategies, monitoring, and maintenance for NFX Chain nodes.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Single Node Deployment](#single-node-deployment)
3. [Multi-Node Cluster](#multi-node-cluster)
4. [Docker Deployment](#docker-deployment)
5. [Systemd Services](#systemd-services)
6. [Monitoring & Alerting](#monitoring--alerting)
7. [Backup & Recovery](#backup--recovery)
8. [Security Hardening](#security-hardening)
9. [Performance Tuning](#performance-tuning)
10. [Upgrades & Maintenance](#upgrades--maintenance)

---

## Prerequisites

### Server Requirements

| Deployment Type | CPU | RAM | Storage | Network |
|-----------------|-----|-----|---------|---------|
| **Testnet/Bootnode** | 2 cores | 4 GB | 50 GB SSD | 100 Mbps |
| **Mainnet Full Node** | 4+ cores | 8+ GB | 200+ GB NVMe | 1 Gbps |
| **Hypernode (Guardian)** | 8 cores | 16+ GB | 500 GB NVMe | 1 Gbps+ |
| **Archive Node** | 8 cores | 32 GB | 1 TB+ NVMe | 1 Gbps |

**Minimum recommended (mainnet):**
- vCPU: 4 cores (Intel Xeon E5 or AMD EPYC)
- RAM: 8 GB DDR4 ECC
- Storage: 200 GB NVMe (provisioned IOPS â‰¥ 3k)
- OS: Ubuntu 22.04 LTS (kernel â‰¥ 5.15)
- Network: Static IP, port 18333/TCP open

### Preliminary Setup

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y
sudo reboot

# 2. Install dependencies
sudo apt install -y \
    build-essential cmake git \
    qtbase5-dev libssl-dev \
    libleveldb-dev libboost-all-dev \
    htop iotop nmon

# 3. Create nfx user (no login, system account)
sudo useradd --system --no-create-home --shell /usr/sbin/nologin nfx

# 4. Create directories
sudo mkdir -p /opt/nfx
sudo mkdir -p /var/lib/nfx/data
sudo mkdir -p /var/log/nfx
sudo chown -R nfx:nfx /opt/nfx /var/lib/nfx /var/log/nfx
```

---

## Single Node Deployment

Quickest way to run a node on a single server.

### Installation Script

```bash
#!/bin/bash
# deploy_single_node.sh

set -e

# Configuration
NFX_VERSION="v1.0.0"
INSTALL_DIR="/opt/nfx"
DATA_DIR="/var/lib/nfx/data"
LOG_DIR="/var/log/nfx"
CONFIG_DIR="/etc/nfx"
NETWORK="testnet"  # or "mainnet"

echo "=== NFX Chain Single Node Deployment ==="

# Download pre-built binaries (or build from source)
echo "Downloading nfx-core..."
wget -q https://github.com/NFXChain/nfx-core/releases/download/${NFX_VERSION}/nfx-core-linux-amd64.tar.gz
tar -xzf nfx-core-linux-amd64.tar.gz -C /tmp/

echo "Downloading nfx-go..."
wget -q https://github.com/NFXChain/nfx-go/releases/download/${NFX_VERSION}/nfx-go-linux-amd64.tar.gz
tar -xzf nfx-go-linux-amd64.tar.gz -C /tmp/

# Install
sudo cp /tmp/nfx-node /opt/nfx/
sudo cp /tmp/nfx-cli /opt/nfx/
sudo chmod +x /opt/nfx/*

# Create directories
sudo mkdir -p "$DATA_DIR" "$LOG_DIR" "$CONFIG_DIR"

# Generate config
sudo tee "$CONFIG_DIR/config.toml" > /dev/null <<EOF
[network]
${NETWORK} = true
p2p_port = 18333
rpc_port = 18332
maxconnections = 50

[rpc]
rpcuser = "nfxuser"
rpcpassword = "$(openssl rand -hex 16)"
rpcallowip = "127.0.0.1"

[consensus]
consensus = "poes"
blocktime = 60
EOF

# Set permissions
sudo chown -R nfx:nfx "$DATA_DIR" "$LOG_DIR"
sudo chmod 600 "$CONFIG_DIR/config.toml"

echo "âœ“ Installation complete"
echo "Config: $CONFIG_DIR/config.toml"
echo "Data: $DATA_DIR"
echo ""
echo "To start node:"
echo "  sudo systemctl start nfx-node"
echo ""
echo "To check status:"
echo "  sudo systemctl status nfx-node"
echo ""
echo "To view logs:"
echo "  sudo journalctl -u nfx-node -f"
```

Run: `sudo bash deploy_single_node.sh`

---

## Multi-Node Cluster

For high availability or load balancing.

### Architecture

```
                    [Load Balancer (HAProxy)]
                             â”‚
                â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                â–¼            â–¼            â–¼
         [API Node 1]  [API Node 2]  [API Node 3]  (nfx-go)
                â”‚            â”‚            â”‚
                â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                             â–¼
                [Hypernode Cluster] (8-12 nodes globally distributed)
                (nfx-core, full validation)
                             â”‚
                â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
                â–¼            â–¼            â–¼
         [LevelDB 1]  [LevelDB 2]  [LevelDB 3]  (Replication)
```

### 1. Hypernode Setup

Hypernodes perform block validation and earn rewards.

**Requirements:**
- Stake â‰¥ 10,000 NFX (locked in contract)
- 24/7 uptime SLA â‰¥ 99.9%
- Geographic distribution (choose underpopulated region)

**Deployment:**

```bash
# On hypernode server
git clone https://github.com/NFXChain/nfx-core.git
cd nfx-core

# Build optimized for server
mkdir build && cd build
cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/nfx \
    -DNFX_HYPERNODE=ON \
    -DNFX_AI_ENABLED=ON

make -j$(nproc)
sudo make install
```

**Hypernode configuration:** `/etc/nfx/hypernode.toml`

```toml
[hypernode]
enabled = true
stake_txid = "abc123..."  # Stake registration transaction
listen = "0.0.0.0:18333"   # P2P port, publicly accessible
rpc_bind = "127.0.0.1:18332"  # RPC only localhost

[ai]
enabled = true
model_path = "/opt/nfx/models/ai_v2.bin"
threshold_valid = 0.85

[performance]
threads = 8              # Use all CPU cores
dbcache = 2048          # 2GB database cache
preload = true          # Preload chain into RAM
```

### 2. Load Balancer (HAProxy)

**`/etc/haproxy/haproxy.cfg`:**

```cfg
global
    log /dev/log local0
    maxconn 2000
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend nfx_rpc
    bind *:18332
    default_backend nfx_api_nodes

backend nfx_api_nodes
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200

    server api01 192.168.1.10:18332 check maxconn 100
    server api02 192.168.1.11:18332 check maxconn 100
    server api03 192.168.1.12:18332 check maxconn 100

    # If all nodes down, queue requests
    option redispatch
```

Start: `sudo systemctl restart haproxy`

### 3. Database Replication (LevelDB)

LevelDB doesn't support built-in replication. Use file-level sync:

```bash
# Option A: DRBD (block device replication)
# Setup DRBD between nodes for synchronous block-level replication

# Option B: rsync snapshot + WAL shipping
# Sync data directory every 5 minutes
*/5 * * * * rsync -az --delete /var/lib/nfx/data/ backup-server:/nfx-backup/

# Option C: Use shared storage (NFS, Ceph)
# Mount shared /var/lib/nfx/data across nodes

# Option D: Orchestrate with Kubernetes (StatefulSet)
# See Docker/K8s section below
```

---

## Docker Deployment

### Multi-Service Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # NFX Core Node (validator)
  nfx-core:
    image: nfxchain/core:${NFX_VERSION:-latest}
    container_name: nfx-core
    restart: unless-stopped
    command: >
      --config /config/config.toml
      --datadir /data
      --log-file /logs/nfx.log
    volumes:
      - ./data:/data
      - ./config:/config
      - ./logs:/logs
    ports:
      - "18333:18333"  # P2P
      - "18332:18332"  # RPC (internal only)
    networks:
      - nfx-internal
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

  # NFX API (Go)
  nfx-api:
    image: nfxchain/api:${NFX_VERSION:-latest}
    container_name: nfx-api
    restart: unless-stopped
    depends_on:
      - nfx-core
    command: >
      --config /config/config.toml
      --rpc-address 0.0.0.0:18332
      --core-address nfx-core:18332
    volumes:
      - ./config:/config
      - ./logs:/logs
    ports:
      - "18332:18332"  # RPC (exposed)
      - "8080:8080"    # REST API
    networks:
      - nfx-internal
      - nfx-external

  # Prometheus metrics exporter
  nfx-exporter:
    image: nfxchain/exporter:latest
    container_name: nfx-exporter
    restart: unless-stopped
    depends_on:
      - nfx-core
    ports:
      - "9100:9100"  # Prometheus metrics endpoint
    networks:
      - nfx-internal

  # Grafana dashboard
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
    environment:
      GF_SECURITY_ADMIN_PASSWORD: "securepassword"
      GF_INSTALL_PLUGINS: "grafana-piechart-panel"
    networks:
      - nfx-external

volumes:
  nfx-data:
  nfx-logs:
  grafana-data:

networks:
  nfx-internal:
    driver: bridge
    internal: true  # No external access
  nfx-external:
    driver: bridge
```

**Start cluster:**

```bash
docker-compose up -d
docker-compose logs -f nfx-api  # Follow logs
```

---

## Systemd Services

Production-ready systemd unit files.

### nfx-node.service

**`/etc/systemd/system/nfx-node.service`:**

```ini
[Unit]
Description=NFX Chain Node
Documentation=https://docs.nfxchain.io
After=network.target network-online.target
Wants=network-online.target

# Restart policy
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
User=nfx
Group=nfx

# Paths
WorkingDirectory=/opt/nfx
ExecStart=/opt/nfx/nfx-node \
    --config /etc/nfx/config.toml \
    --datadir /var/lib/nfx/data \
    --log-file /var/log/nfx/nfx.log
ExecReload=/bin/kill -HUP $MAINPID

# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/nfx /var/log/nfx /opt/nfx

# Resource limits
LimitNOFILE=65535
LimitNPROC=4096
MemoryLimit=4G
MemoryMax=4G
CPUQuota=300%

# Restart policy
Restart=on-failure
RestartSec=5s

# Runtime directory (PID file)
RuntimeDirectory=nfx
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

**Enable and start:**

```bash
sudo systemctl daemon-reload
sudo systemctl enable nfx-node
sudo systemctl start nfx-node
sudo systemctl status nfx-node
```

### nfx-wallet.service

**`/etc/systemd/system/nfx-wallet.service`:**

```ini
[Unit]
Description=NFX Wallet (Desktop)
After=graphical.target network.target
Wants=network.target

[Service]
Type=simple
User=user  # Regular desktop user
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/user/.Xauthority

ExecStart=/opt/nfx/nfx-wallet --minimized
Restart=on-failure

# Allow GUI access
PrivateTmp=false

[Install]
WantedBy=graphical.target
```

---

## Monitoring & Alerting

### 1. Prometheus Metrics Exporter

NFX exposes metrics on port 9100 by default:

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'nfx'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:18332']  # RPC metrics
```

**Available metrics:**

| Metric | Type | Description |
|--------|------|-------------|
| `nfx_blocks` | gauge | Current block height |
| `nfx_peers` | gauge | Connected peers count |
| `nfx_mempool_size` | gauge | Transactions in mempool |
| `nfx_difficulty` | gauge | Current PoW/PoES difficulty |
| `nfx_utxo_count` | gauge | UTXO set size |
| `nfx_bandwidth_in` | counter | Inbound P2P bytes |
| `nfx_bandwidth_out` | counter | Outbound P2P bytes |
| `nfx_ai_score_avg` | gauge | Average AI block score |
| `nfx_staking_total` | gauge | Total staked NFX |

### 2. Grafana Dashboard

Import dashboard `NFX_Chain_Overview.json`:

```json
{
  "dashboard": {
    "title": "NFX Chain Node",
    "panels": [
      {
        "title": "Block Height",
        "targets": [{"expr": "nfx_blocks"}],
        "type": "graph"
      },
      {
        "title": "Peer Count",
        "targets": [{"expr": "nfx_peers"}],
        "type": "stat"
      },
      {
        "title": "Mempool Size",
        "targets": [{"expr": "nfx_mempool_size"}],
        "type": "graph"
      },
      {
        "title": "Inbound/Outbound Traffic",
        "targets": [
          {"expr": "rate(nfx_bandwidth_in_bytes[5m])"},
          {"expr": "rate(nfx_bandwidth_out_bytes[5m])"}
        ],
        "type": "graph"
      }
    ]
  }
}
```

### 3. Alertmanager Rules

```yaml
# alert.rules.yml
groups:
  - name: nfx_node
    rules:
      - alert: NodeDown
        expr: up{job="nfx"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "NFX node is down"

      - alert: LowPeers
        expr: nfx_peers < 8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low peer count ({{ $value }})"

      - alert: NotSynced
        expr: nfx_verification_progress < 0.999
        for: 10m
        labels:
          severity: warning
```

**Integrations:**
- **Email:** `smtp` config in Alertmanager
- **Slack:** Incoming webhook
- **Discord:** Webhook bot
- **PagerDuty:** API integration

---

## Backup & Recovery

### 1. Blockchain Data

**Do NOT backup `chainstate/` or `blocks/` folders** â€” they can be re-downloaded.

**NEED to backup:**
- `wallet/` directory (encrypted wallet files)
- `config.toml` (RPC credentials, custom settings)
- `mnemonic.txt` or private keys (if exported)

### 2. Wallet Backup

```bash
# Backup wallet database
tar -czf ~/nfx-wallet-backup-$(date +%Y%m%d).tar.gz \
    ~/.nfx/wallets/

# Include config
tar -rf ~/nfx-wallet-backup-$(date +%Y%m%d).tar.gz \
    ~/.nfx/config.toml

# Encrypt backup
gpg --armor --encrypt --recipient your@email.com \
    ~/nfx-wallet-backup-*.tar.gz
```

**Automated backup script:**

```bash
#!/bin/bash
# backup_nfx.sh - Daily wallet backup

BACKUP_DIR="/backup/nfx"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="nfx-wallet-$DATE.tar.gz"
ENCRYPTED_FILE="$BACKUP_FILE.gpg"

# Create backup
tar -czf /tmp/$BACKUP_FILE \
    /home/nfx/.nfx/wallets/ \
    /etc/nfx/config.toml

# Encrypt with GPG
gpg --armor --encrypt \
    --recipient backup@nfxchain.io \
    --output $BACKUP_DIR/$ENCRYPTED_FILE \
    /tmp/$BACKUP_FILE

# Upload to cloud (optional)
# aws s3 cp $BACKUP_DIR/$ENCRYPTED_FILE s3://nfx-backups/

# Cleanup
rm -f /tmp/$BACKUP_FILE

echo "âœ“ Backup saved: $ENCRYPTED_FILE"
```

### 3. Disaster Recovery

**Scenario: Server failure**

```bash
# 1. Provision new server (same OS, architecture)
# 2. Install nfx-core, nfx-go binaries
# 3. Restore wallet backup
tar -xzf nfx-wallet-backup.tar.gz -C ~/
# Enter wallet passphrase when prompted

# 4. Restore config
sudo cp config.toml /etc/nfx/config.toml

# 5. Start node
sudo systemctl start nfx-node

# 6. Wait for sync (from scratch or from recent snapshot)
tail -f /var/log/nfx/nfx.log
```

**Note:** Chainstate (block index, UTXO set) must be rebuilt from scratch or restored from a recent snapshot.

---

## Security Hardening

### 1. Firewall Configuration

```bash
# Ubuntu UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow 22/tcp

# Allow P2P (18333) from anywhere (required)
sudo ufw allow 18333/tcp

# Allow RPC only from localhost or trusted IPs
sudo ufw allow from 127.0.0.1 to any port 18332
sudo ufw allow from 192.168.1.0/24 to any port 18332

# Deny all others
sudo ufw --force enable
```

### 2. SSH Hardening

```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no  # Use key-based only
MaxAuthTries 3
AllowUsers nfx admin
```

### 3. AppArmor/SELinux Profile

**`/etc/apparmor.d/usr.bin.nfx-node`:**

```apparmor
#include <tunables/global>

/opt/nfx/nfx-node {
  #include <abstractions/base>

  # Read-only access
  /etc/nfx/config.toml r,
  /usr/lib/** mr,

  # Data directory (read-write)
  /var/lib/nfx/data/** rw,
  /var/log/nfx/** rw,

  # Network
  network inet stream,
  network inet dgram,

  # Capabilities
  capability net_bind_service,
  capability setuid,
  capability setgid,
}
```

Enable: `sudo apparmor_parser -r /etc/apparmor.d/usr.bin.nfx-node`

### 4. Transaction Monitoring (Fraud Detection)

```go
// Use WebSocket to monitor suspicious transactions
client.Subscribe("rawtx", func(msg ws.Message) {
    tx := decodeTx(msg.Data["hex"])

    // Check for anomalies
    if tx.VinCount > 100 {
        alert("Massive transaction (100+ inputs)")
    }
    if tx.FeeRate < MIN_FEE_RATE {
        alert("Undercut fee attack")
    }
    if isDoubleSpend(tx) {
        alert("Double spend attempt!")
    }
})
```

### 5. RPC Rate Limiting

```toml
[rpc]
rate_limit = 100      # Max requests/second
rate_limit_burst = 200  # Burst allowance
```

### 6. Regular Updates

```bash
# Automated security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Keep nfx-chain updated (manual check weekly)
# Check: https://github.com/NFXChain/nfx-chain/releases
```

---

## Performance Tuning

### 1. Database Cache

**`config.toml`:**

```toml
[database]
dbcache = 4096  # 4GB cache for 16GB RAM system
```

**Rule of thumb:**
- `dbcache` = 50% of RAM for dedicated node
- `dbcache` = 25% of RAM for node + wallet on same machine

### 2. Parallel Script Execution

```toml
[vm]
max_parallel_scripts = 8  # Number of CPU cores
```

### 3. LevelDB Optimization

Edit `src/storage/leveldb_opt.cpp` (recompile needed):

```cpp
leveldb::Options opts;
opts.compression = leveldb::kSnappyCompression;  // Fast compression
opts.write_buffer_size = 64 * 1024 * 1024;       // 64MB write buffer
opts.max_open_files = 1000;                      // Keep more files open
opts.target_file_size_base = 64 * 1024 * 1024;  // 64MB SSTables
opts.max_file_opening_threads = 4;              // Parallel file open
```

### 4. Network Buffers

```bash
# Increase TCP buffers
sudo sysctl -w net.core.rmem_max=268435456
sudo sysctl -w net.core.wmem_max=268435456
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 268435456"
sudo sysctl -w net.ipv4.tcp_wmem="4096 16384 268435456"

# Persist in /etc/sysctl.d/99-nfx.conf
```

### 5. Huge Pages (Linux)

```bash
# Allocate huge pages (requires root)
echo 1024 | sudo tee /proc/sys/vm/nr_hugepages

# Pin process to NUMA node 0
numactl --cpunodebind=0 --membind=0 /opt/nfx/nfx-node
```

### 6. Disable Swap (if enough RAM)

```bash
sudo swapoff -a
# Remove swap entry from /etc/fstab
```

---

## Upgrades & Maintenance

### 1. Planned Maintenance (Mainnet)

**Maintenance window schedule:**

| Activity | Frequency | Duration | Impact |
|----------|-----------|----------|--------|
| Minor updates (security) | As needed | 5 min | Brief RPC downtime |
| Major version upgrade | Quarterly | 30 min | Node restart required |
| Database compaction | Weekly (off-peak) | 1-2 hrs | High I/O, no downtime if preloading |
| Log rotation | Daily | <1 sec | None |

### 2. Upgrade Procedure

```bash
#!/bin/bash
# upgrade_nfx.sh

set -e

echo "Stopping node..."
sudo systemctl stop nfx-node

# Backup wallet and config
echo "Backing up wallet..."
BACKUP_DIR="/backup/pre-upgrade/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR
cp -r /var/lib/nfx/wallets/ "$BACKUP_DIR/"
cp /etc/nfx/config.toml "$BACKUP_DIR/"

# Download new version
echo "Downloading new version..."
VERSION="v1.1.0"
wget -q "https://github.com/NFXChain/nfx-core/releases/download/${VERSION}/nfx-core-linux-amd64.tar.gz"
wget -q "https://github.com/NFXChain/nfx-go/releases/download/${VERSION}/nfx-go-linux-amd64.tar.gz"

# Replace binaries
sudo tar -xzf nfx-core-linux-amd64.tar.gz -C /opt/nfx/
sudo tar -xzf nfx-go-linux-amd64.tar.gz -C /opt/nfx/
sudo chown nfx:nfx /opt/nfx/nfx-*

# Verify new version
/opt/nfx/nfx-node --version

# Start node
echo "Starting node..."
sudo systemctl start nfx-node

# Wait for sync
echo "Waiting for node to become ready..."
for i in {1..60}; do
    if nfx-cli getblockcount | grep -q "^[0-9]"; then
        echo "âœ“ Node is responding"
        break
    fi
    sleep 2
done

# Verify chain sync
CURRENT=$(nfx-cli getblockcount)
echo "Current block: $CURRENT"

echo "âœ“ Upgrade complete!"
```

### 3. Rollback

If upgrade fails:

```bash
# Restore previous version
sudo cp $BACKUP_DIR/nfx-node /opt/nfx/
sudo cp $BACKUP_DIR/nfx-cli /opt/nfx/

# Restart
sudo systemctl restart nfx-node
```

### 4. Database Reindex (Corruption Repair)

```bash
# Stop node
sudo systemctl stop nfx-node

# Backup current state
cp -r /var/lib/nfx/data /var/lib/nfx/data.backup

# Reindex chainstate (fast)
nfx-node --reindex-chainstate

# Or full reindex (slower, downloads all blocks again)
nfx-node --reindex

# Start node after reindex completes
sudo systemctl start nfx-node
```

---

*Next: [Configuration Reference](configuration.md) | [FAQ](../faq.md)*
