# Code Examples

Practical examples for using NFX Chain components.

## Table of Contents

1. [Go: Simple Balance Checker](#go-simple-balance-checker)
2. [Go: Send Transaction](#go-send-transaction)
3. [Go: Wallet Management](#go-wallet-management)
4. [Go: Monitor Mempool](#go-monitor-mempool)
5. [Go: WebSocket Events](#go-websocket-events)
6. [C: Basic Node Control](#c-basic-node-control)
7. [C: Transaction Signing](#c-transaction-signing)
8. [Python: RPC Queries](#python-rpc-queries)
9. [Rust: Build Signed Transaction](#rust-build-signed-transaction)
10. [Docker: Full Node](#docker-full-node)
11. [Shell: CLI Utilities](#shell-cli-utilities)
12. [JavaScript: REST API](#javascript-rest-api)

---

## Go: Simple Balance Checker

Check balance of multiple addresses concurrently:

```go
package main

import (
    "context"
    "fmt"
    "log"
    "sync"
    "time"

    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    addresses := []string{
        "NFX1abc123...",
        "NFX1def456...",
        "NFX1ghi789...",
    }

    // Create RPC client
    client := rpc.NewClient(
        "http://localhost:18332",
        rpc.WithCredentials("nfxuser", "yourpassword"),
        rpc.WithTimeout(10*time.Second),
    )

    // Concurrent balance queries
    var wg sync.WaitGroup
    results := make(map[string]uint64)

    for _, addr := range addresses {
        wg.Add(1)
        go func(address string) {
            defer wg.Done()
            balance, err := client.GetBalance(address)
            if err != nil {
                log.Printf("Error for %s: %v", address, err)
                return
            }
            results[address] = balance
        }(addr)
    }

    wg.Wait()

    // Print results
    fmt.Println("\nBalances:")
    fmt.Println("========================")
    for addr, bal := range results {
        fmt.Printf("%s: %d satoshis (%.8f NFX)\n",
            addr, bal, float64(bal)/1e8)
    }
}
```

**Output:**
```
Balances:
========================
NFX1abc123...: 150000000 satoshis (1.50000000 NFX)
NFX1def456...: 75000000 satoshis (0.75000000 NFX)
NFX1ghi789...: 0 satoshis (0.00000000 NFX)
```

---

## Go: Send Transaction

Complete example with error handling and confirmation:

```go
package main

import (
    "encoding/hex"
    "fmt"
    "log"
    "time"

    "github.com/NFXChain/nfx-go/rpc"
    "github.com/NFXChain/nfx-go/sdk/txbuilder"
)

func main() {
    // 1. Connect to node
    client := rpc.NewClient(
        "http://localhost:18332",
        rpc.WithCredentials("nfxuser", "password123"),
        rpc.WithTimeout(30*time.Second),
    )

    // 2. Get current nonce for sender
    addr := "NFX1sender_address..."
    txs, err := client.ListUnspent(addr, 1, 9999999)
    if err != nil {
        log.Fatal("Failed to get UTXOs:", err)
    }

    if len(txs) == 0 {
        log.Fatal("No UTXOs available for spending")
    }

    // 3. Build transaction
    builder := txbuilder.New(client).
        From(addr).
        To("NFX1receiver_address...", 1.5*1e8).  // 1.5 NFX
        SetFee(0.001 * 1e8).                     // 0.001 NFX fee
        SetChange(addr)                          // Send change back

    // Add all available UTXOs as inputs
    for _, utxo := range txs {
        builder.AddInput(utxo.TxID, utxo.Vout, utxo.Amount)
    }

    rawTx, err := builder.Build()
    if err != nil {
        log.Fatal("Build failed:", err)
    }

    fmt.Printf("Raw transaction (hex): %s\n", hex.EncodeToString(rawTx))

    // 4. Sign transaction (if not pre-signed by node)
    // If node has wallet enabled, just broadcast raw hex
    // If external signing, use wallet.Sign() here

    // 5. Broadcast
    txid, err := client.SendRawTransaction(rawTx)
    if err != nil {
        log.Fatal("Broadcast failed:", err)
    }

    fmt.Printf("\nâœ“ Transaction broadcast successfully!\n")
    fmt.Printf("  TXID: %s\n", txid)
    fmt.Printf("  Explorer: https://explorer.testnet.nfxchain.io/tx/%s\n", txid)

    // 6. Wait for confirmation
    fmt.Println("\nWaiting for first confirmation...")
    for i := 0; i < 60; i++ {
        tx, err := client.GetTransaction(txid)
        if err == nil && tx.Confirmations >= 1 {
            fmt.Printf("âœ“ Confirmed in block %d after %d seconds\n",
                tx.BlockHeight, i+1)
            return
        }
        time.Sleep(1 * time.Second)
    }

    fmt.Println("âš  Still unconfirmed after 60 seconds")
}
```

**Key points:**
- `ListUnspent` finds available UTXOs
- `txbuilder` handles change calculation automatically
- Always set a fee (default 0.001 NFX)
- Wait for at least 1 confirmation for small amounts, 6 for large

---

## Go: Wallet Management (HD Wallet)

Create and manage hierarchical deterministic wallet:

```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/NFXChain/nfx-go/sdk/hdwallet"
    "github.com/NFXChain/nfx-go/sdk/encoder"
    "github.com/NFXChain/nfx-go/rpc"
)

func main() {
    // 1. Generate new mnemonic (12 words, BIP-39)
    mnemonic, err := hdwallet.GenerateMnemonic(128)  // 128 bits = 12 words
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("=== NEW WALLET CREATED ===")
    fmt.Println("âš ï¸  IMPORTANT: Write down your mnemonic:")
    fmt.Println("â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€")
    fmt.Println(mnemonic)
    fmt.Println("â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€")
    fmt.Println("\nNever share this with anyone!")

    // 2. Save mnemonic securely
    err = os.WriteFile("wallet.mnemonic", []byte(mnemonic), 0600)
    if err != nil {
        log.Fatal("Failed to save mnemonic:", err)
    }

    // 3. Derive wallet from mnemonic
    wallet, err := hdwallet.FromMnemonic(mnemonic)
    if err != nil {
        log.Fatal(err)
    }

    // 4. Derive first external address (BIP-44 path)
    // NFX Chain: purpose=44' (0x8000002C), coin_type=777' (0x80000309)
    key, err := wallet.Derive("m/44'/777'/0'/0/0")
    if err != nil {
        log.Fatal(err)
    }

    address := key.Address()
    fmt.Printf("\nFirst address: %s\n", address)

    // 5. Derive more addresses
    addresses := make([]string, 10)
    for i := 0; i < 10; i++ {
        key, _ := wallet.Derive(fmt.Sprintf("m/44'/777'/0'/0/%d", i))
        addresses[i] = key.Address()
    }

    fmt.Println("\nFirst 10 addresses:")
    for i, addr := range addresses {
        fmt.Printf("%2d: %s\n", i, addr)
    }

    // 6. Connect to node and request funds (testnet faucet)
    client := rpc.NewClient(
        "http://localhost:18332",
        rpc.WithCredentials("nfxuser", "password"),
    )

    // Request from testnet faucet
    fmt.Println("\nRequesting from testnet faucet...")
    // faucetTXID, err := client.RequestFromFaucet(addresses[0])
    // fmt.Println("Faucet TX:", faucetTXID)

    // Import first address into node wallet
    // client.ImportAddress(addresses[0], "My Test Wallet")

    // 7. Check balance after 1 minute
    time.Sleep(60 * time.Second)
    balance, err := client.GetBalance(addresses[0])
    if err != nil {
        log.Println("Balance check failed:", err)
    } else {
        fmt.Printf("\nBalance: %d satoshis (%.8f NFX)\n",
            balance, float64(balance)/1e8)
    }

    // 8. Generate WIF (Wallet Import Format) for backup
    wif := key.WIF()
    fmt.Printf("\nWIF (for backup): %s\n", wif)

    // NEVER commit mnemonic/WIF to git!
    // Use environment variables or keychain in production
}
```

**Recover from mnemonic:**

```go
// Recover existing wallet
recoveredWallet, err := hdwallet.FromMnemonic(
    "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about"
)
if err != nil {
    log.Fatal(err)
}

// Derive same address
key, _ := recoveredWallet.Derive("m/44'/777'/0'/0/0")
fmt.Println("Recovered address:", key.Address())
// Same as: NFX1abc123...
```

---

## Go: Monitor Mempool

Real-time mempool monitoring with notifications:

```go
package main

import (
    "encoding/hex"
    "fmt"
    "log"
    "time"

    "github.com/NFXChain/nfx-go/rpc/ws"
)

func main() {
    // Connect WebSocket
    client, err := ws.NewClient("ws://localhost:18334")
    if err != nil {
        log.Fatal("WS connection failed:", err)
    }
    defer client.Close()

    fmt.Println("Connected to WebSocket feed")

    // Subscribe to raw transaction announcements
    client.Subscribe("rawtx", func(msg ws.Message) {
        fmt.Printf("\n[Raw TX] Received\n")

        // Decode hex to see details
        txHex, _ := hex.DecodeString(msg.Data["hex"].(string))
        fmt.Printf("  Size: %d bytes\n", len(txHex))
        fmt.Printf("  Raw: %s...\n", hex.EncodeToString(txHex)[:64])

        // Parse to extract basics (requires tx decoding)
        // You can use client.RPC().DecodeRawTransaction() for details
    })

    // Also subscribe to specific address
    myAddr := "NFX1myaddress..."
    client.Subscribe("address."+myAddr, func(msg ws.Message) {
        fmt.Printf("\n[TX for %s]\n", myAddr)
        fmt.Printf("  TXID: %s\n", msg.Data["txid"])
        fmt.Printf("  Amount: %s\n", msg.Data["amount"])
        fmt.Printf("  Confirmations: %d\n", msg.Data["confirmations"])
    })

    // Keep running
    fmt.Println("Monitoring mempool... (Ctrl+C to stop)")
    select {}
}
```

**Output example:**
```
Connected to WebSocket feed
Monitoring mempool... (Ctrl+C to stop)

[Raw TX] Received
  Size: 225 bytes
  Raw: 0200000001a3b2...

[TX for NFX1myaddress...]
  TXID: 7f83b1657ff1fc53b92dc18648a1d865...
  Amount: 1.50000000
  Confirmations: 0
```

---

## Go: WebSocket Events

Subscribe to live blockchain events:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"

    "github.com/NFXChain/nfx-go/rpc/ws"
)

type EventHandler func(event string, data map[string]interface{})

func main() {
    wsClient, err := ws.NewClient(
        "ws://localhost:18334",
        ws.WithReconnect(true),      // Auto-reconnect
        ws.WithReconnectInterval(5), // 5 second delay
    )
    if err != nil {
        log.Fatal(err)
    }
    defer wsClient.Close()

    // Enable all event types you need
    events := []string{
        "block",           // New block
        "tx",              // Transaction in block
        "rawtx",           // Raw mempool tx
        "alert",           // Network alert
        "ai_score",        // AI governance results
    }

    for _, event := range events {
        wsClient.Subscribe(event, func(msg ws.Message) {
            handleEvent(event, msg.Data)
        })
    }

    // Blocking call with reconnect
    wsClient.Listen()
}

func handleEvent(event string, data map[string]interface{}) {
    switch event {
    case "block":
        height := int(data["height"].(float64))
        hash := data["hash"].(string)
        fmt.Printf("ðŸ“¦ New block #%d: %s\n", height, hash)

    case "tx":
        txid := data["txid"].(string)
        blockHash := data["blockhash"].(string)
        fmt.Printf("ðŸ’° TX %s confirmed in block %s\n", txid, blockHash)

    case "rawtx":
        txid := data["txid"].(string)
        fmt.Printf("ðŸ“¨ Mempool TX: %s\n", txid)

    case "ai_score":
        blockHash := data["block_hash"].(string)
        score := data["score"].(map[string]interface{})
        fmt.Printf("ðŸ§  AI Score for %s: valid=%.2f, suspicious=%.2f\n",
            blockHash,
            score["valid"],
            score["suspicious"])

    default:
        // Unexpected event
        j, _ := json.MarshalIndent(data, "", "  ")
        fmt.Printf("Event %s: %s\n", event, j)
    }
}
```

**Real-time monitoring example:**

```go
// Track specific address
monitorAddress := "NFX1myaddress..."
client.Subscribe("address."+monitorAddress, func(msg ws.Message) {
    switch msg.Type {
    case "receive":
        fmt.Printf("âœ“ Received %s NFX\n", msg.Data["amount"])
    case "send":
        fmt.Printf("âœ— Sent %s NFX\n", msg.Data["amount"])
    case "stake":
        fmt.Printf("ðŸ“ˆ Staking reward: %s NFX\n", msg.Data["amount"])
    }
})
```

---

## C: Basic Node Control

Using C bindings to control node lifecycle:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "nfx_bindings.h"

void print_blockchain_info(nfx_context_t* ctx);
void print_peer_info(nfx_context_t* ctx);

int main() {
    printf("NFX Chain Node Control\n");
    printf("=======================\n\n");

    // 1. Create context
    nfx_context_t* ctx = nfx_create();
    if (!ctx) {
        fprintf(stderr, "Failed to create context\n");
        return 1;
    }

    // 2. Initialize with config
    const char* config_path = "config.toml";
    nfx_error_t err = nfx_init(ctx, config_path);
    if (err != NFX_OK) {
        fprintf(stderr, "Init failed: %s\n",
                nfx_error_string(err));
        nfx_destroy(ctx);
        return 1;
    }

    printf("âœ“ Initialized successfully\n\n");

    // 3. Show current state
    print_blockchain_info(ctx);
    print_peer_info(ctx);

    // 4. Start node
    printf("Starting node...\n");
    err = nfx_node_start(ctx);
    if (err != NFX_OK) {
        fprintf(stderr, "Failed to start: %s\n",
                nfx_error_string(err));
        nfx_destroy(ctx);
        return 1;
    }
    printf("âœ“ Node started (PID: %d)\n\n",
           nfx_node_get_pid(ctx));

    // 5. Wait for user
    printf("Press ENTER to stop node...\n");
    getchar();

    // 6. Graceful shutdown
    printf("Stopping node...\n");
    nfx_node_stop(ctx);

    // 7. Wait for shutdown
    nfx_node_wait(ctx);
    printf("âœ“ Node stopped\n");

    // 8. Cleanup
    nfx_destroy(ctx);
    printf("âœ“ Context destroyed\n");

    return 0;
}

void print_blockchain_info(nfx_context_t* ctx) {
    nfx_blockchain_info_t info;
    nfx_error_t err = nfx_get_blockchain_info(ctx, &info);

    if (err == NFX_OK) {
        printf("Chain: %s\n", info.chain);
        printf("Blocks: %lld\n", info.blocks);
        printf("Headers: %lld\n", info.headers);
        printf("Best block: %s\n", info.best_block_hash);
        printf("Difficulty: %.8f\n", info.difficulty);
        printf("Progress: %.2f%%\n", info.verification_progress * 100);
        printf("\n");
    } else {
        printf("Failed to get chain info: %s\n",
               nfx_error_string(err));
    }
}

void print_peer_info(nfx_context_t* ctx) {
    nfx_peer_info_t* peers = NULL;
    size_t count = 0;

    nfx_error_t err = nfx_get_peer_info(ctx, &peers, &count);
    if (err == NFX_OK && count > 0) {
        printf("Connected peers (%zu):\n", count);
        for (size_t i = 0; i < count; i++) {
            printf("  %s - %s\n",
                   peers[i].address,
                   peers[i].version);
        }
        printf("\n");
        nfx_free(peers);
    }
}
```

**Compile and run:**

```bash
gcc -o node_control node_control.c -lnfx -L. -I.
./node_control
```

---

## C: Transaction Signing

Sign a transaction offline (cold storage):

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "nfx_bindings.h"

// Sign transaction without broadcasting (cold storage)
int sign_offline_tx(
    nfx_context_t* ctx,
    const char* raw_tx_hex,
    const char* wif_private_key,
    char* signed_tx_hex,
    size_t* signed_len
) {
    // 1. Import private key temporarily (memory only, not saved)
    nfx_error_t err = nfx_import_privkey(ctx, wif_private_key, "");
    if (err != NFX_OK) {
        fprintf(stderr, "Import failed: %s\n",
                nfx_error_string(err));
        return -1;
    }

    // 2. Sign raw transaction
    err = nfx_sign_raw_transaction(
        ctx,
        raw_tx_hex,
        NULL,   // No prev txs needed for simple SigHash
        signed_tx_hex,
        signed_len
    );

    // 3. Remove key from memory (cold storage security)
    nfx_remove_privkey(ctx, wif_private_key);

    if (err != NFX_OK) {
        fprintf(stderr, "Sign failed: %s\n",
                nfx_error_string(err));
        return -1;
    }

    return 0;
}

int main() {
    // Unsigned transaction (created elsewhere)
    const char* unsigned_tx = "0200000001a3b2...";

    // Private key in WIF (never commit this!)
    const char* wif = "L1aW4aubDFB7yfras2S1mN3bqg9w7YjY72K9Z6fM8iJ234567890";

    char signed_tx[1024];
    size_t signed_len = sizeof(signed_tx);

    if (sign_offline_tx(
        ctx,
        unsigned_tx,
        wif,
        signed_tx,
        &signed_len
    ) == 0) {
        printf("Signed transaction: %.*s\n",
               (int)signed_len, signed_tx);

        // Copy to air-gapped machine via USB
        // Then broadcast from online machine
    }

    return 0;
}
```

---

## Python: RPC Queries

Using `requests` for simple queries:

```python
#!/usr/bin/env python3
"""
NFX Chain RPC Example in Python
Requires: pip install requests
"""

import requests
import json
import base64

class NFXClient:
    def __init__(self, rpc_url, rpc_user, rpc_password):
        self.url = rpc_url
        self.auth = (rpc_user, rpc_password)

    def call(self, method, params=None):
        payload = {
            "jsonrpc": "2.0",
            "id": "pyclient",
            "method": method,
            "params": params or []
        }

        resp = requests.post(
            self.url,
            auth=self.auth,
            json=payload,
            timeout=10
        )
        resp.raise_for_status()
        data = resp.json()

        if "error" in data and data["error"] is not None:
            raise Exception(f"RPC Error: {data['error']}")

        return data["result"]

    def get_balance(self, address):
        return self.call("getbalance", [address])

    def get_block_count(self):
        return self.call("getblockcount")

    def get_block(self, hash_or_height, verbose=1):
        return self.call("getblock", [hash_or_height, verbose])

    def send_transaction(self, from_addr, to_addr, amount, fee=0.001):
        return self.call("sendtransaction", [{
            "from": from_addr,
            "to": to_addr,
            "amount": amount,
            "fee": fee
        }])

# Usage
if __name__ == "__main__":
    client = NFXClient(
        "http://localhost:18332",
        "nfxuser",
        "yourpassword"
    )

    # Get info
    info = client.call("getblockchaininfo")
    print(f"Chain: {info['chain']}, Blocks: {info['blocks']}")

    # Get balance
    balance = client.get_balance("NFX1abc123...")
    print(f"Balance: {balance} satoshis ({balance/1e8:.8f} NFX)")

    # Send transaction
    # txid = client.send_transaction(
    #     "NFX1sender...",
    #     "NFX1receiver...",
    #     1.5  # NFX
    # )
    # print(f"Sent! TXID: {txid}")
```

---

## Rust: Build Signed Transaction

Create and sign transaction using Rust bindings:

```rust
use nfx_bindings::{nfx_create, nfx_init, nfx_get_balance, nfx_send_transaction, nfx_destroy, nfx_error_t};
use std::ffi::{CString, CStr};
use std::os::raw::{c_char, c_void};

struct NFXClient {
    ctx: *mut c_void,
}

impl NFXClient {
    fn new(config: &str) -> Result<Self, String> {
        let ctx = unsafe { nfx_create() };
        let c_config = CString::new(config).unwrap();

        let err = unsafe { nfx_init(ctx, c_config.as_ptr()) };
        if err != 0 {
            unsafe { nfx_destroy(ctx) };
            return Err(format!("Init failed: {}", err));
        }

        Ok(Self { ctx })
    }

    fn get_balance(&self, address: &str) -> Result<u64, String> {
        let c_addr = CString::new(address).unwrap();
        let mut balance: u64 = 0;

        let err = unsafe {
            nfx_get_balance(
                self.ctx,
                c_addr.as_ptr(),
                &mut balance as *mut u64
            )
        };

        if err == 0 {
            Ok(balance)
        } else {
            Err(format!("Get balance failed: {}", err))
        }
    }

    fn send(&self, from: &str, to: &str, amount: u64) -> Result<String, String> {
        // ... build and send transaction
        unimplemented!()
    }
}

fn main() {
    let client = NFXClient::new("config.toml").unwrap();

    let balance = client.get_balance("NFX1abc...").unwrap();
    println!("Balance: {} satoshis", balance);

    // Build transaction
    // Sign with wallet
    // Broadcast
}
```

---

## Docker: Full Node

Run node with Docker Compose:

```yaml
# docker-compose.yml
version: '3.8'

services:
  nfx-node:
    image: nfxchain/node:latest
    container_name: nfx-node
    restart: unless-stopped
    ports:
      - "18332:18332"   # RPC
      - "18333:18333"   # P2P
      - "18334:18334"   # WebSocket
    volumes:
      - ./data:/root/.nfx
      - ./config:/etc/nfx
    command: >
      --config /etc/nfx/config.toml
      --testnet
      --log-file=/root/.nfx/logs/nfx.log
    environment:
      - TZ=America/Sao_Paulo

networks:
  default:
    name: nfx-network
```

Build custom image:

```dockerfile
# Dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    nfx-core \
    nfx-go \
    && rm -rf /var/lib/apt/lists/*

COPY config.toml /etc/nfx/config.toml
COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

EXPOSE 18332 18333 18334

ENTRYPOINT ["/entrypoint.sh"]
```

Run:
```bash
docker-compose up -d
docker-compose logs -f nfx-node
```

---

## Shell: CLI Utilities

### Check Node Health

```bash
#!/bin/bash
# check_nfx.sh - Monitor NFX node health

NODE_URL="http://localhost:18332"
USER="nfxuser"
PASS="password"

# Check if process is running
if ! pgrep -x "nfx-node" > /dev/null; then
    echo "âŒ Node is not running"
    exit 2
fi

# Get blockchain info
info=$(curl -s -u $USER:$PASS \
    -H "Content-Type: application/json" \
    -d '{"jsonrpc":"1.0","id":"health","method":"getblockchaininfo","params":[]}' \
    $NODE_URL)

blocks=$(echo $info | jq -r '.result.blocks')
headers=$(echo $info | jq -r '.result.headers')
progress=$(echo $info | jq -r '.result.verificationprogress')

if (( $(echo "$blocks < $headers" | bc -l) )); then
    echo "âš ï¸  Syncing: $blocks/$headers blocks ($(echo "$progress*100" | bc)%)"
else
    echo "âœ“ Synced at block $blocks"
fi

# Check peers
peers=$(curl -s -u $USER:$PASS \
    -d '{"jsonrpc":"1.0","id":"peer","method":"getpeerinfo","params":[]}' \
    $NODE_URL | jq '.result | length')

if [ "$peers" -lt 8 ]; then
    echo "âš ï¸  Low peer count: $peers/50"
else
    echo "âœ“ Connected to $peers peers"
fi

# Check RPC latency
start=$(date +%s%N)
curl -s -u $USER:$PASS -o /dev/null $NODE_URL > /dev/null
end=$(date +%s%N)
latency=$(( (end - start) / 1000000 ))
echo "RPC latency: ${latency}ms"
```

### Broadcast Transaction

```bash
#!/bin/bash
# broadcast_tx.sh - Send raw transaction

TX_HEX="$1"

if [ -z "$TX_HEX" ]; then
    echo "Usage: $0 <hex-tx>"
    exit 1
fi

RESPONSE=$(curl -s -u nfxuser:password \
    -H "Content-Type: application/json" \
    -d "{\"jsonrpc\":\"2.0\",\"id\":\"broadcast\",\"method\":\"sendrawtransaction\",\"params\":[\"$TX_HEX\"]}" \
    http://localhost:18332/)

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    echo "âŒ Failed:"
    echo "$RESPONSE" | jq '.error'
    exit 1
else
    echo "âœ“ Broadcasted!"
    echo "$RESPONSE" | jq '.result'
fi
```

### Monitor New Blocks

```bash
#!/bin/bash
# watch_blocks.sh - Poll for new blocks

LAST_BLOCK=$(nfx-cli getblockcount)
echo "Current block: $LAST_BLOCK"
echo "Watching for new blocks... (Ctrl+C to stop)"

while true; do
    sleep=5
    CURRENT=$(nfx-cli getblockcount)

    if [ "$CURRENT" -gt "$LAST_BLOCK" ]; then
        echo "$(date +%H:%M:%S) - New block: $CURRENT"
        LAST_BLOCK=$CURRENT
        sleep=1  # Fast poll immediately after new block
    fi

    sleep $sleep
done
```

---

## JavaScript: REST API

Consume NFX Chain API from browser/Node.js:

```javascript
// nfx-client.js
const NFXClient = class {
    constructor(baseURL, username, password) {
        this.baseURL = baseURL;
        this.auth = 'Basic ' + Buffer.from(
            `${username}:${password}`
        ).toString('base64');
    }

    async call(method, params = []) {
        const response = await fetch(this.baseURL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': this.auth
            },
            body: JSON.stringify({
                jsonrpc: '2.0',
                id: 'js-client',
                method,
                params
            })
        });

        const data = await response.json();
        if (data.error) {
            throw new Error(data.error.message);
        }
        return data.result;
    }

    async getBalance(address) {
        return this.call('getbalance', [address]);
    }

    async getBlockCount() {
        return this.call('getblockcount');
    }

    async getBlock(hash) {
        return this.call('getblock', [hash, 2]);  // verbose=2
    }

    async sendTransaction(from, to, amount, fee = 0.001) {
        return this.call('sendtransaction', [{
            from, to, amount, fee
        }]);
    }
};

// Usage in Node.js
if (require.main === module) {
    const client = new NFXClient(
        'http://localhost:18332',
        'nfxuser',
        'password'
    );

    (async () => {
        try {
            const info = await client.getBlockchainInfo();
            console.log('Blocks:', info.blocks);

            const balance = await client.getBalance('NFX1...');
            console.log('Balance:', balance / 1e8, 'NFX');
        } catch (err) {
            console.error('Error:', err.message);
        }
    })();
}

module.exports = NFXClient;
```

**Frontend example (React):**

```jsx
import { useState, useEffect } from 'react';
import NFXClient from './nfx-client';

function NFXBalance({ address }) {
    const [balance, setBalance] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const client = new NFXClient(
            process.env.REACT_APP_NFX_RPC_URL,
            process.env.REACT_APP_NFX_USER,
            process.env.REACT_APP_NFX_PASS
        );

        client.getBalance(address)
            .then(bal => setBalance(bal / 1e8))
            .catch(console.error)
            .finally(() => setLoading(false));
    }, [address]);

    if (loading) return <div>Loading...</div>;
    return <div>Balance: {balance.toFixed(8)} NFX</div>;
}
```

---

## Automated Transaction Sender

Batch send to multiple recipients:

```go
package main

import (
    "bufio"
    "encoding/csv"
    "fmt"
    "log"
    "os"
    "time"

    "github.com/NFXChain/nfx-go/rpc"
    "github.com/NFXChain/nfx-go/sdk/txbuilder"
)

type Payment struct {
    Address string
    Amount  float64  // NFX
    Label   string
}

func main() {
    // Load payments from CSV
    file, err := os.Open("payments.csv")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    reader := csv.NewReader(bufio.NewReader(file))
    records, err := reader.ReadAll()
    if err != nil {
        log.Fatal(err)
    }

    var payments []Payment
    for _, row := range records {
        payments = append(payments, Payment{
            Address: row[0],
            Amount:  parseFloat(row[1]),
            Label:   row[2],
        })
    }

    fmt.Printf("Processing %d payments...\n", len(payments))

    // Connect
    client := rpc.NewClient(
        "http://localhost:18332",
        rpc.WithCredentials("nfxuser", "pass"),
    )

    sender := "NFX1source..."
    total := 0.0

    for i, p := range payments {
        fmt.Printf("[%d/%d] Sending %.8f NFX to %s\n",
            i+1, len(payments), p.Amount, p.Address)

        // Build and send
        txid, err := client.SendTransaction(&rpc.SendTxRequest{
            From:   sender,
            To:     p.Address,
            Amount: p.Amount * 1e8,
            Fee:    0.001 * 1e8,
        })

        if err != nil {
            log.Printf("Failed payment %d: %v\n", i+1, err)
            continue
        }

        fmt.Printf("  âœ“ Sent! TXID: %s\n", txid)
        total += p.Amount

        // Rate limit: 1 tx per 2 seconds
        time.Sleep(2 * time.Second)
    }

    fmt.Printf("\nâœ“ Completed: %.8f NFX sent in %d transactions\n",
        total, len(payments))
}

func parseFloat(s string) float64 {
    var f float64
    fmt.Sscanf(s, "%f", &f)
    return f
}
```

**payments.csv:**
```
address,amount,label
NFX1alice...,1.5,Invoice #001
NFX1bob...,0.75,Refund
NFX1carol...,2.0,Payment
```

---

## Using Docker for Testing

```bash
#!/bin/bash
# spin_testnet.sh - Spin up local test network with Docker

cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  node1:
    image: nfxchain/node:latest
    container_name: nfx-node-1
    command: --testnet --rpcport=18332 --p2pport=18333 --datadir=/data --rpcuser=nfxuser --rpcpassword=test123
    ports:
      - "18332:18332"
      - "18333:18333"
    volumes:
      - node1_data:/data
    networks:
      nfxnet:

  node2:
    image: nfxchain/node:latest
    container_name: nfx-node-2
    command: --testnet --rpcport=18334 --p2pport=18335 --datadir=/data --rpcuser=nfxuser --rpcpassword=test123 --connect=nfx-node-1:18333
    depends_on:
      - node1
    volumes:
      - node2_data:/data
    networks:
      nfxnet:

volumes:
  node1_data:
  node2_data:

networks:
  nfxnet:
    driver: bridge
EOF

docker-compose up -d

echo "Waiting for nodes to start..."
sleep 10

# Check status
curl -s -u nfxuser:test123 \
  -d '{"jsonrpc":"1.0","id":"test","method":"getblockchaininfo","params":[]}' \
  http://localhost:18332/ | jq .

# Stop
# docker-compose down
```

---

## Shell: Quick Balance Script

One-liner to check balance from terminal:

```bash
#!/bin/bash
# nfx-balance.sh <address>
# Usage: ./nfx-balance.sh NFX1abc...

ADDRESS="$1"
RPC_URL="http://localhost:18332"
USER="nfxuser"
PASS="password"

if [ -z "$ADDRESS" ]; then
    echo "Usage: $0 <NFX_address>"
    exit 1
fi

BALANCE=$(curl -s -u $USER:$PASS \
    -H "Content-Type: application/json" \
    -d "{\"jsonrpc\":\"1.0\",\"id\":\"balance\",\"method\":\"getbalance\",\"params\":[\"$ADDRESS\"]}" \
    $RPC_URL | jq -r '.result')

if [ "$BALANCE" = "null" ]; then
    echo "Error: Address not found or RPC error"
    exit 1
fi

NFX=$(echo "scale=8; $BALANCE/100000000" | bc)
echo "Balance: $BALANCE satoshis ($NFX NFX)"
```

---

## Generating QR Codes (Bash + Python)

```bash
#!/bin/bash
# nfx-qr.sh - Generate QR code for address

ADDRESS="$1"

if [ -z "$ADDRESS" ]; then
    echo "Usage: $0 <NFX_address>"
    exit 1
fi

# Generate QR code PNG (requires qrencode)
qrencode -o /tmp/nfx_qr.png -s 256 "$ADDRESS"

echo "QR code saved to /tmp/nfx_qr.png"
echo "Address: $ADDRESS"

# Open with default viewer (Linux)
xdg-open /tmp/nfx_qr.png &
```

**Python alternative (cross-platform):**

```python
import qrcode
import sys

addr = sys.argv[1]
img = qrcode.make(addr)
img.save("nfx_qr.png")
print(f"QR saved: nfx_qr.png for {addr}")
```

---

## Monitoring Script (Golang)

Health check with alerting:

```go
package main

import (
    "fmt"
    "log"
    "net/smtp"
    "time"

    "github.com/NFXChain/nfx-go/rpc"
)

type Monitor struct {
    client *rpc.Client
    alerts []string
}

func (m *Monitor) Check() error {
    // 1. Node reachable?
    info, err := m.client.GetBlockchainInfo()
    if err != nil {
        return fmt.Errorf("node unreachable: %w", err)
    }

    // 2. Synced?
    if info.Headers > info.Blocks {
        m.alerts = append(m.alerts,
            fmt.Sprintf("Node syncing: %d/%d blocks",
                info.Blocks, info.Headers))
    }

    // 3. Peer count
    peers, _ := m.client.GetPeerInfo()
    if len(peers) < 8 {
        m.alerts = append(m.alerts,
            fmt.Sprintf("Low peers: %d (want â‰¥8)", len(peers)))
    }

    // 4. Mempool size
    mempool, _ := m.client.GetMempoolInfo()
    if mempool.Size > 1000 {
        m.alerts = append(m.alerts,
            fmt.Sprintf("High mempool: %d txs", mempool.Size))
    }

    return nil
}

func (m *Monitor) SendAlert() {
    if len(m.alerts) == 0 {
        return
    }

    body := "NFX Node Alert:\n\n"
    for _, a := range m.alerts {
        body += "- " + a + "\n"
    }

    // Send email (configure SMTP)
    auth := smtp.PlainAuth("", "user@gmail.com", "pass", "smtp.gmail.com")
    to := []string{"admin@example.com"}
    msg := []byte("To: admin@example.com\r\n" +
        "Subject: NFX Node Alert\r\n" +
        "\r\n" + body + "\r\n")

    smtp.SendMail("smtp.gmail.com:587", auth, "node@nfx", to, msg)
}

func main() {
    client := rpc.NewClient("http://localhost:18332",
        rpc.WithCredentials("nfxuser", "pass"),
    )

    monitor := &Monitor{client: client}

    for {
        err := monitor.Check()
        if err != nil {
            log.Println("Check failed:", err)
        }

        if len(monitor.alerts) > 0 {
            monitor.SendAlert()
            fmt.Println("Alerts sent:", monitor.alerts)
        } else {
            fmt.Println("All systems OK")
        }

        time.Sleep(60 * time.Second)
    }
}
```

---

## Testing with Testnet Faucet

Get testnet NFX:

```bash
# Request from official faucet
curl -X POST https://faucet.testnet.nfxchain.io/api/request \
    -H "Content-Type: application/json" \
    -d '{"address": "NFX1your_address"}'

# Or use nfx-cli (if wallet RPC enabled)
nfx-cli --testnet getnewaddress
nfx-cli --testnet getbalance  # Should show 0 initially

# Request from Discord bot
# Join https://discord.gg/nfx
# Use /faucet <address> command in #testnet-faucet channel
```

---

*Next: [Deployment Guide](../guides/deployment.md) | [Configuration Reference](../guides/configuration.md) | [API Reference](../api/)*
