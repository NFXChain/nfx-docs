# nfx-go Module Documentation

**Module:** `nfx-go`  
**Language:** Go 1.19+  
**Type:** Go Module + Binaries  
**Path:** `nfx-go/`  

## Overview

`nfx-go` provides the high-level Go API, command-line tools, and reference node implementation for NFX Chain. It wraps the C bindings (`libnfx.so`) with idiomatic Go interfaces, concurrency support, and production-ready client/server components.

## Directory Structure

```
nfx-go/
â”œâ”€â”€ go.mod                 # Go module definition
â”œâ”€â”€ go.sum                 # Dependencies lockfile
â”œâ”€â”€ README.md
â”‚
â”œâ”€â”€ api/                  # JSON-RPC server (node implementation)
â”‚   â”œâ”€â”€ main.go          # Entry point: nfx-node
â”‚   â”œâ”€â”€ server.go        # HTTP/REST server
â”‚   â”œâ”€â”€ handler.go       # RPC method handlers
â”‚   â”œâ”€â”€ middleware.go    # Auth, rate limiting, logging
â”‚   â””â”€â”€ ws.go            # WebSocket notifications
â”‚
â”œâ”€â”€ cli/                  # Command-line tool (nfx-cli)
â”‚   â”œâ”€â”€ main.go
â”‚   â”œâ”€â”€ commands.go      # CLI command definitions
â”‚   â”œâ”€â”€ balance.go
â”‚   â”œâ”€â”€ send.go
â”‚   â”œâ”€â”€ block.go
â”‚   â””â”€â”€ wallet.go
â”‚
â”œâ”€â”€ rpc/                  # JSON-RPC client library
â”‚   â”œâ”€â”€ client.go        # RPC client implementation
â”‚   â”œâ”€â”€ types.go         # Request/response structs
â”‚   â”œâ”€â”€ auth.go          # HTTP basic auth
â”‚   â””â”€â”€ notifications.go # WebSocket event subscriptions
â”‚
â”œâ”€â”€ sdk/                  # Go SDK (library)
â”‚   â”œâ”€â”€ wallet.go        # Wallet management
â”‚   â”œâ”€â”€ transaction.go   # Transaction builder
â”‚   â”œâ”€â”€ signer.go        // Transaction signing (ECDSA/SPHINCS+)
â”‚   â”œâ”€â”€ encoder.go       // Address/base58 encoding
â”‚   â””â”€â”€ mnemonic.go      // BIP-39 mnemonic generation
â”‚
â”œâ”€â”€ wallet/              # Wallet implementation (optional)
â”‚   â”œâ”€â”€ app.go          // Full wallet application
â”‚   â”œâ”€â”€ database.go     // SQLite wallet DB
â”‚   â”œâ”€â”€ keystore.go     // Encrypted key storage
â”‚   â””â”€â”€ txn_monitor.go  // Transaction tracking
â”‚
â””â”€â”€ internal/           // Internal utilities
    â”œâ”€â”€ config/         // TOML config parsing
    â”œâ”€â”€ crypto/         // Go crypto wrappers
    â”œâ”€â”€ encoding/       // Hex, base58, base64
    â””â”€â”€ logging/        // Structured logging (zap)
```

## Installation

### From Source

```bash
cd nfx-go

# Download dependencies
go mod download

# Build nfx-node (API server)
go build -o nfx-node ./api

# Build nfx-cli (command-line tool)
go build -o nfx-cli ./cli

# Install to $GOPATH/bin
go install ./api
go install ./cli

# Verify
./nfx-node --version
./nfx-cli --help
```

### Using go install (latest release)

```bash
go install github.com/NFXChain/nfx-go/api@latest
go install github.com/NFXChain/nfx-go/cli@latest

# Binaries installed to $GOPATH/bin
which nfx-node  # ~/go/bin/nfx-node
```

## Quick Start

### 1. Run a Node

```bash
# Copy example config
cp config.example.toml config.toml
nano config.toml  # Edit RPC credentials, ports, etc.

# Start node (foreground)
./nfx-node --config config.toml

# Start node (background)
./nfx-node --config config.toml --daemon

# Check logs
tail -f ~/.nfx/logs/nfx.log

# Check status
./nfx-cli getinfo
```

### 2. Using the RPC Client (Library)

```go
package main

import (
    "fmt"
    "log"
    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    // Connect to node
    client := rpc.NewClient(
        "http://localhost:18332",  // RPC URL
        rpc.WithCredentials("nfxuser", "yourpassword"),
        rpc.WithTimeout(30),
    )

    // Get blockchain info
    info, err := client.GetBlockchainInfo()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Chain: %s, Blocks: %d\n", info.Chain, info.Blocks)

    // Get balance
    balance, err := client.GetBalance("NFX123abc...")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Balance: %d satoshis\n", balance)

    // Send transaction
    txID, err := client.SendTransaction(&rpc.SendTxRequest{
        From:      "NFX_sender_address",
        To:        "NFX_receiver_address",
        Amount:    1000000, // 0.01 NFX (assuming 8 decimals)
        Fee:       1000,
        Nonce:     1,
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("TXID: %s\n", txID)
}
```

### 3. Using nfx-cli

```bash
# Get node info
./nfx-cli getinfo

# Get balance
./nfx-cli getbalance NFX123...

# Send funds
./nfx-cli send \
  --from=NFX_sender \
  --to=NFX_receiver \
  --amount=1.5 \
  --fee=0.001

# List peers
./nfx-cli listpeers

# Get block by hash
./nfx-cli getblock 0000000000000000000...

# Create wallet (encrypted)
./nfx-cli walletcreate "My Wallet"

# Import from mnemonic
./nfx-cli walletimportmnemonic "word1 word2 ... word12"
```

## nfx-node (API Server)

### Command-Line Options

```bash
nfx-node [OPTIONS]

Options:
  --config PATH          Load config from PATH (default: ~/.nfx/config.toml)
  --datadir PATH         Data directory (default: ~/.nfx/data)
  --logfile PATH         Log file path
  --log-level LEVEL      Log level: debug|info|warn|error (default: info)
  --daemon               Run as daemon (background)
  --console              Log to console (default: true unless --daemon)
  --testnet              Use testnet (override config)
  --mainnet              Use mainnet (override config)
  --develop              Enable developer mode (fewer confirmations)
  --help, -h             Show help
  --version              Show version
```

**Examples:**

```bash
# Quick testnet node
nfx-node --testnet --daemon

# Custom config
nfx-node --config=/etc/nfx/mainnet.toml

# Debug mode
nfx-node --log-level=debug --console

# View version
nfx-node --version
# Output: NFX Node version 1.0.0 (commit: abc123)
```

### Configuration

`nfx-node` reads TOML config files. See full reference in [installation guide](../guides/installation.md#configuration).

```bash
# Config locations (checked in order):
1. Path from --config flag
2. $HOME/.nfx/config.toml
3. /etc/nfx/config.toml
4. ./config.toml (current directory)
```

**Minimal config for testing:**

```toml
# ~/.nfx/config.toml
[network]
testnet = true
rpc_port = 18332
p2p_port = 18333

[rpc]
rpcuser = "nfxuser"
rpcpassword = "test123"

[consensus]
consensus = "poes"
```

### API Endpoints

#### HTTP JSON-RPC

**Endpoint:** `POST http://localhost:18332/`

Headers:
```
Content-Type: application/json
Authorization: Basic <base64(nfxuser:password)>
```

Request format:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getblockchaininfo",
  "params": []
}
```

Response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "chain": "testnet",
    "blocks": 12345,
    "headers": 12345,
    "bestblockhash": "000000000000...",
    "difficulty": 1.00000000,
    "verificationprogress": 0.9999
  }
}
```

**Common RPC Methods:**

| Method | Params | Description |
|--------|--------|-------------|
| `getblockchaininfo` | none | General blockchain info |
| `getblockcount` | none | Current block height |
| `getblock` | hash, verbose? | Get block by hash |
| `gettransaction` | txid | Get transaction details |
| `getbalance` | address | Get address balance |
| `sendtransaction` | from, to, amount, fee | Send funds |
| `getrawmempool` | none | List mempool txids |
| `getpeerinfo` | none | Connected peers |
| `getmininginfo` | none | Mining status & difficulty |
| `getnetworkinfo` | none | Network stats |

### REST API (Experimental)

If `[api]` enabled in config:

```
GET    /api/v1/blocks/:hash
GET    /api/v1/blocks/height/:height
GET    /api/v1/transactions/:txid
GET    /api/v1/address/:address/balance
POST   /api/v1/transactions
GET    /api/v1/peers
```

Example:
```bash
curl http://localhost:8080/api/v1/blocks/height/100
# Returns block JSON
```

### WebSocket Notifications

Connect to `ws://localhost:18334/` (if `rpc.ws_port` enabled):

```go
package main

import (
    "github.com/NFXChain/nfx-go/rpc/ws"
)

func main() {
    client := ws.NewClient("ws://localhost:18334")
    defer client.Close()

    // Subscribe to new blocks
    client.Subscribe("block", func(msg ws.Message) {
        fmt.Printf("New block: %v\n", msg.Data)
    })

    // Subscribe to transactions affecting your address
    client.Subscribe("address.NFX123...", func(msg ws.Message) {
        fmt.Printf("TX for monitored address: %v\n", msg.Data)
    })

    // Keep connection alive
    select {}
}
```

## nfx-cli (Command-Line Tool)

### Command Reference

```bash
# General info
nfx-cli getinfo              # Node status summary
nfx-cli getnetworkinfo       # Network statistics
nfx-cli getmininginfo        # Mining statistics

# Blockchain
nfx-cli getblockcount        # Current height
nfx-cli getblock HASH        # Get block details
nfx-cli getblockhash HEIGHT  # Get block hash at height
nfx-cli getchaintxstats      # Chain transaction stats

# Transactions
nfx-cli getbalance ADDRESS   # Account balance
nfx-cli listunspent          # List UTXOs
nfx-cli send FROM TO AMOUNT  # Send funds
nfx-cli decoderawtransaction HEX  # Decode hex tx
nfx-cli signrawtransaction HEX [keys...]  # Sign tx

# Wallet management
nfx-cli walletcreate "name"          # Create encrypted wallet
nfx-cli walletlist                  # List wallets
nfx-cli walletimportmnemonic MNEMONIC # Import from seed
nfx-cli dumpprivkey ADDRESS          # Export private key (encrypted)
nfx-cli importprivkey WIF            # Import WIF key

# Peers
nfx-cli listpeers           # Connected peers
nfx-cli addnode ADDRESS     # Add persistent peer
nfx-cli disconnnode PEERID  # Disconnect peer

# Mining (if PoW enabled)
nfx-cli generatetoaddress N ADDRESS  # Mine N blocks
nfx-cli getmininginfo       # Mining status

# Governance
nfx-cli aiscore BLOCKHASH   # Get AI score for block
nfx-cli aistatus            # AI engine status

# Configuration
nfx-cli help [command]      # Show command help
nfx-cli version             # Version info
```

### Examples

**Send 2.5 NFX:**
```bash
nfx-cli send \
  --from=NFX_sender_123... \
  --to=NFX_receiver_456... \
  --amount=2.5 \
  --fee=0.001 \
  --comment="Payment for services"
```

**Check balance:**
```bash
nfx-cli getbalance NFX123abc456...
```

**List UTXOs:**
```bash
nfx-cli listunspent --minconf=1 --maxconf=999999
```

Output:
```json
[
  {
    "txid": "abc123...",
    "vout": 0,
    "address": "NFX123...",
    "amount": 1.50000000,
    "confirmations": 5,
    "spendable": true
  }
]
```

**View block:**
```bash
nfx-cli getblock 0000000000000000000000000000000000000000000000000000000000000000 2
```
`2` = verbose (include full transaction data)

## Go SDK (Library)

The `nfx-go/sdk` package provides high-level abstractions.

### Wallet Manager

```go
package main

import (
    "github.com/NFXChain/nfx-go/sdk/wallet"
    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    // 1. Create RPC client
    client := rpc.NewClient(
        "http://localhost:18332",
        rpc.WithCredentials("nfxuser", "password"),
    )

    // 2. Create wallet manager
    wm := wallet.NewManager(client, wallet.WithDBPath("./wallet.db"))

    // 3. Create new wallet (BIP-39 mnemonic)
    mnemonic, err := wm.CreateWallet("MyWallet", "strong-passphrase")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Mnemonic (SAVE THIS!):", mnemonic)

    // 4. Get first address
    addr, err := wm.GetAddress(0)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Address:", addr)

    // 5. Get balance
    bal, err := wm.GetBalance(addr)
    fmt.Printf("Balance: %d satoshis\n", bal)

    // 6. Send transaction
    txHash, err := wm.Send(addr, "recipient_address", 1.5, 0.001)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("TX sent:", txHash)
}
```

### Transaction Builder

```go
package main

import (
    "github.com/NFXChain/nfx-go/sdk/txbuilder"
    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    client := rpc.NewClient("http://localhost:18332")

    // Create transaction builder
    builder := txbuilder.New(client)

    tx, err := builder.
        From("sender_address").
        AddOutput("recipient_address", 1.5*1e8). // 1.5 NFX
        SetFee(0.001 * 1e8).
        SetNonce(1).
        Build()

    if err != nil {
        log.Fatal(err)
    }

    // Sign transaction (with wallet)
    signedTx, err := wallet.Sign(tx)
    if err != nil {
        log.Fatal(err)
    }

    // Broadcast
    txid, err := client.Broadcast(signedTx)
    fmt.Println("TXID:", txid)
}
```

### HD Wallet (BIP-44)

```go
import "github.com/NFXChain/nfx-go/sdk/hdwallet"

// Create from mnemonic
wallet, err := hdwallet.FromMnemonic(
    "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about"
)
if err != nil {
    log.Fatal(err)
}

// Derive NFX Chain path (BIP-44)
// purpose=44' (0x8000002C), coin_type=777' (0x80000309),
// account=0', change=0, address_index=0
key, err := wallet.Derive("m/44'/777'/0'/0/0")
if err != nil {
    log.Fatal(err)
}

// Get address (P2PKH)
address := key.Address()
fmt.Println("Address:", address)

// Get private key (WIF format)
wif := key.WIF()
fmt.Println("WIF:", wif)

// Sign message
sig, err := key.Sign([]byte("Hello NFX Chain"))
fmt.Println("Signature:", sig)
```

## Advanced Usage

### Custom HTTP Client

```go
client := rpc.NewClient(
    "https://node.example.com:18332",
    rpc.WithCredentials("user", "pass"),
    rpc.WithTimeout(60*time.Second),
    rpc.WithRetry(3),              // Retry failed requests
    rpc.WithRetryDelay(2*time.Second),
    rpc.WithTLSConfig(&tls.Config{
        InsecureSkipVerify: false, // For self-signed certs
    }),
)
```

### Concurrent Requests

```go
// Fire multiple requests in parallel
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(idx int) {
        defer wg.Done()
        bal, _ := client.GetBalance(fmt.Sprintf("NFX%03d", idx))
        fmt.Printf("Addr %03d: %d\n", idx, bal)
    }(i)
}
wg.Wait()
```

### Event Subscriptions (WebSocket)

```go
wsClient, _ := rpc.NewWebSocket("ws://localhost:18334")

// Subscribe to new blocks
wsClient.Subscribe("block", func(msg *rpc.Notification) {
    fmt.Printf("New block %d: %s\n",
        msg.Data["height"],
        msg.Data["hash"])
})

// Subscribe to address monitoring
wsClient.Subscribe("address.NFX123...", func(msg *rpc.Notification) {
    fmt.Println("Transaction for my address!")
    fmt.Printf("TX: %v\n", msg.Data)
})

// Subscribe to mempool
wsClient.Subscribe("rawtx", func(msg *rpc.Notification) {
    fmt.Println("New transaction in mempool")
})

// Unsubscribe
wsClient.Unsubscribe("block")
```

### Context Cancellation (Timeouts)

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

balance, err := client.GetBalanceCtx(ctx, "NFX123...")
if err != nil {
    if errors.Is(err, context.DeadlineExceeded) {
        fmt.Println("Request timed out")
    } else {
        fmt.Println("Error:", err)
    }
}
```

### Error Handling

```go
balance, err := client.GetBalance("NFX123...")
if err != nil {
    var rpcErr *rpc.RPCError
    if errors.As(err, &rpcErr) {
        // Detailed RPC error
        fmt.Printf("Code: %d, Message: %s\n",
            rpcErr.Code, rpcErr.Message)
    } else if errors.Is(err, rpc.ErrConnectionFailed) {
        fmt.Println("Node not running or unreachable")
    } else if errors.Is(err, rpc.ErrUnauthorized) {
        fmt.Println("Invalid RPC credentials")
    } else {
        fmt.Println("Unknown error:", err)
    }
    return
}
```

## Performance Considerations

### Connection Pool

The RPC client uses an HTTP connection pool:

```go
client := rpc.NewClient(
    "http://localhost:18332",
    rpc.WithMaxIdleConns(100),
    rpc.WithMaxConnsPerHost(50),
    rpc.WithIdleConnTimeout(90*time.Second),
)
```

### Request Batching (Future)

Batch multiple RPC calls into single HTTP request:

```go
batch := client.NewBatch()
batch.Call("getbalance", "ADDR1")
batch.Call("getbalance", "ADDR2")
batch.Call("getblockcount")
batch.Call("getmininginfo")

results := batch.Execute()
```

### Parallel Transaction Submission

```go
// Submit many transactions concurrently
txns := generateTransactions(1000)

sem := make(chan struct{}, 10) // Limit to 10 concurrent
var wg sync.WaitGroup
for _, tx := range txns {
    wg.Add(1)
    go func(t *sdk.Transaction) {
        defer wg.Done()
        sem <- struct{}{}
        defer func() { <-sem }()

        err := client.SendTransaction(t)
        if err != nil {
            log.Println("Failed:", err)
        }
    }(tx)
}
wg.Wait()
```

## Testing

### Unit Tests

```bash
cd nfx-go
go test ./rpc/...        # Test RPC package
go test -v              # Verbose output
go test -run TestGetInfo # Run specific test
go test -cover          # Coverage report
go test -bench=.        # Benchmarks
```

**Example test:**

```go
package rpc_test

import (
    "testing"
    "github.com/NFXChain/nfx-go/rpc"
)

func TestGetBlockchainInfo(t *testing.T) {
    client := rpc.NewClient("http://localhost:18332")
    info, err := client.GetBlockchainInfo()
    if err != nil {
        t.Fatalf("Failed: %v", err)
    }

    if info.Chain == "" {
        t.Error("Chain name empty")
    }
    if info.Blocks < 0 {
        t.Error("Negative block count")
    }
}
```

### Integration Tests

```bash
# Start test node
nfx-node --testnet --datadir=/tmp/test-nfx &
sleep 5

# Run integration tests
go test -tags=integration ./rpc/...

# Cleanup
kill $(pgrep nfx-node)
rm -rf /tmp/test-nfx
```

## Benchmarking

```go
package main

import (
    "fmt"
    "time"
    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    client := rpc.NewClient("http://localhost:18332")

    // Benchmark GetBlockchainInfo
    iterations := 1000
    start := time.Now()
    for i := 0; i < iterations; i++ {
        client.GetBlockchainInfo()
    }
    elapsed := time.Since(start)

    fmt.Printf("%d requests in %v\n", iterations, elapsed)
    fmt.Printf("Avg: %v/request\n", elapsed/time.Duration(iterations))
}
```

Expected output on decent hardware:
```
1000 requests in 2.5s
Avg: 2.50ms/request
```

## Troubleshooting

### Connection Refused

```bash
# Check if node is running
ps aux | grep nfx-node

# Check port
netstat -tulpn | grep 18332

# Start node
nfx-node --config ~/.nfx/config.toml
```

### Authentication Failed

```bash
# Verify credentials in config.toml
cat ~/.nfx/config.toml | grep rpcuser
cat ~/.nfx/config.toml | grep rpcpassword

# Update if needed, then restart node
nfx-cli stop
nfx-node --config ~/.nfx/config.toml
```

### Out of Memory

```bash
# Reduce database cache in config.toml
[database]
dbcache = 128  # Reduce from 256MB to 128MB

# Or limit Go heap
export GODEBUG=gctrace=1
./nfx-node --maxheap=512mb
```

### High CPU Usage

```bash
# Enable AI governance?
# Disable if not needed
[ai]
enabled = false

# Or reduce AI scan frequency
[ai]
scan_interval = "5m"  # Default: "1m"
```

## Profiling

### CPU Profiling

```bash
# Enable profiling
nfx-node --profile=cpu --profile-path=/tmp/profile.out

# Analyze
go tool pprof /tmp/profile.out
(pprof) top
(pprof) web  # Generate graph.svg
```

### Memory Profiling

```bash
nfx-node --profile=mem --profile-path=/tmp/mem.out
go tool pprof -alloc_space /tmp/mem.out
```

### Trace (Execution Timeline)

```bash
nfx-node --trace=/tmp/trace.out
go tool trace /tmp/trace.out
# Opens in browser
```

## Contributing

### Code Style

- Use `gofmt` and `goimports`
- Run `go vet ./...` before PR
- Add tests for new features
- Update documentation

```bash
# Format code
gofmt -w .

# Organize imports
goimports -w .

# Lint
go vet ./...

# Run tests
go test ./...
```

### Submitting Changes

1. Fork repository
2. Create feature branch
3. Implement change with tests
4. Ensure `go test ./...` passes
5. Submit PR with clear description

## License

MIT License â€” See project root for details.

---

*Next: [Wallet Module](wallet.md) | [API Reference../docs/api/) | [Examples../docs/examples/)*
