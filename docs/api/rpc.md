# JSON-RPC API Reference

NFX Chain node exposes JSON-RPC 2.0 interface for external clients.

## Connection

**URL:** `http://localhost:18332/` (testnet) or `http://localhost:8332/` (mainnet)

**Authentication:** HTTP Basic Auth with `rpcuser` and `rpcpassword` from config.

**Content-Type:** `application/json`

---

## Request Format

```json
{
  "jsonrpc": "2.0",
  "id": "unique_id",
  "method": "method_name",
  "params": [arg1, arg2, ...]
}
```

## Response Format

**Success:**
```json
{
  "jsonrpc": "2.0",
  "id": "unique_id",
  "result": { ... }
}
```

**Error:**
```json
{
  "jsonrpc": "2.0",
  "id": "unique_id",
  "error": {
    "code": -1,
    "message": "Error description"
  }
}
```

---

## Standard Methods

### Blockchain

#### `getblockchaininfo`

Returns an object containing various state info regarding blockchain processing.

**Params:** None

**Result:**
```json
{
  "chain": "testnet",
  "blocks": 12345,
  "headers": 12345,
  "bestblockhash": "000000000000...",
  "difficulty": 1.00000000,
  "verificationprogress": 0.9999,
  "size_on_disk": 52428800,
  "pruned": false
}
```

#### `getblockcount`

Returns the number of blocks in the best block chain.

**Result:** `12345`

#### `getblock`

Returns block data.

**Params:**
- `"hash"` â€” Block hash (string)
- `verbose` â€” 0 (hex) | 1 (txids) | 2 (full tx data) (number, optional, default=1)

**Example:**
```bash
curl -u nfxuser:password -X POST http://localhost:18332/ \
  -d '{"jsonrpc":"2.0","id":"getblock","method":"getblock","params":["abc123...",2]}'
```

#### `getblockhash`

Returns hash of block at given height.

**Params:** `height` (number)

**Result:** Hex block hash string

#### `getbestblockhash`

Returns hash of best (tip) block.

**Result:** Hex string

---

### Transactions

#### `getrawtransaction`

Returns raw transaction data.

**Params:**
- `txid` (string)
- `verbose` (number, optional, default=0)

#### `sendrawtransaction`

Broadcasts raw transaction hex.

**Params:** `hexstring` (string)

**Result:** `"txid"` (string)

#### `sendtransaction`

High-level send (creates and broadcasts).

**Params:**
```json
{
  "from": "NFX1sender...",
  "to": "NFX1receiver...",
  "amount": 1.5,
  "fee": 0.001
}
```

**Result:** `"txid"` (string)

#### `decoderawtransaction`

Decode hex to JSON.

**Params:** `hexstring` (string)

---

### Balances & UTXOs

#### `getbalance`

Returns total balance for an address.

**Params:**
- `address` (string)
- `minconf` (number, optional, default=1)
- `maxconf` (number, optional)

**Result:** `150000000` (satoshis)

#### `listunspent`

List unspent transaction outputs.

**Params:**
- `minconf` (number, default=1)
- `maxconf` (number, default=9999999)
- `["addresses"]` (array of strings, optional filter)

**Result:**
```json
[
  {
    "txid": "abc123...",
    "vout": 0,
    "address": "NFX1...",
    "amount": 1.5,
    "confirmations": 5,
    "scriptPubKey": "76a914...",
    "spendable": true
  }
]
```

---

### Peer & Network

#### `getnetworkinfo`

Returns network-related info.

**Result:**
```json
{
  "version": 100000,
  "subversion": "/NFX:1.0.0/",
  "connections": 42,
  "connections_in": 35,
  "connections_out": 7,
  "networkactive": true,
  "relayfee": 0.001,
  "inbound_bw": 1024000,
  "outbound_bw": 512000
}
```

#### `getpeerinfo`

List connected peers.

**Result:** Array of peer objects

---

### Mining (if PoW enabled)

#### `getmininginfo`

```json
{
  "blocks": 12345,
  "currentblockweight": 0,
  "currentblocktx": 0,
  "difficulty": 1.0,
  "networkhashps": 1000000,
  "pooledtx": 15,
  "chain": "testnet"
}
```

#### `generatetoaddress`

Mine blocks to address (regtest/signet only).

**Params:** `nblocks`, `address`

---

### Wallet RPC (if wallet enabled)

#### `getwalletinfo`

```json
{
  "walletname": "MyWallet",
  "walletversion": 169900,
  "balance": 150000000.00,
  "unconfirmed_balance": 0.00,
  "immature_balance": 0.00,
  "txcount": 5,
  "keypoolsize": 100,
  "keypoolsize_hd_internal": 100,
  "unlocked_until": 0
}
```

#### `listwallets`

Returns array of loaded wallet names.

#### `sendtoaddress`

Send funds (wallet RPC).

**Params:** `address`, `amount`, `comment`, `comment_to`

#### `dumpprivkey`

Export private key (requires wallet unlock).

**âš ï¸ Security:** Use only for backup/recovery. Never share.

---

### Governance & AI

#### `gethypernodes`

List registered guardian/hypernodes.

**Params:** `include_status` (bool, optional)

#### `getaiscore`

Get AI governance score for block.

**Params:** `block_hash` (string)

**Result:**
```json
{
  "block_hash": "abc123...",
  "valid": 0.92,
  "suspicious": 0.05,
  "malicious": 0.03,
  "recommended_action": "accept"
}
```

---

## Notifications (WebSocket)

Connect to `ws://localhost:18334/` (if enabled).

### Subscribe

```json
{
  "jsonrpc": "2.0",
  "id": "sub1",
  "method": "subscribe",
  "params": ["block"]
}
```

**Supported channels:**
- `"block"` â€” New blocks
- `"tx"` â€” Transactions included in blocks
- `"rawtx"` â€” Mempool transactions
- `"alert"` â€” Network alerts
- `"ai_score"` â€” AI governance results

### Notification Format

```json
{
  "jsonrpc": "2.0",
  "method": "block",
  "params": {
    "hash": "000000...",
    "height": 12345,
    "timestamp": 1704067200
  }
}
```

---

## Error Codes

| Code | Message | Meaning |
|------|---------|---------|
| -1  | "Internal error" | Generic |
| -2  | "Invalid parameter" | Bad argument |
| -3  | "RPC method not found" | Unknown method |
| -4  | "Invalid address" | Malformed address |
| -5  | "Insufficient funds" | Balance too low |
| -6  | "Invalid transaction" | Consensus violation |
| -7  | "JSON decode error" | Malformed request |
| -8  | "Wallet error" | Wallet-specific |
| -9  | "Unauthorized" | Bad RPC credentials |
| -10 | "Not found" | Item doesn't exist |

---

## Code Samples

### cURL

```bash
# Get block count
curl -u nfxuser:password \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":"count","method":"getblockcount","params":[]}' \
  http://localhost:18332/

# Send transaction
curl -u nfxuser:password \
  -d '{"jsonrpc":"2.0","id":"send","method":"sendtransaction","params":[{"from":"NFX1...","to":"NFX1...","amount":1.5,"fee":0.001}]}' \
  http://localhost:18332/
```

### Python (requests)

```python
import requests
import json

r = requests.post(
    "http://localhost:18332/",
    auth=("nfxuser", "password"),
    json={"jsonrpc":"2.0","id":1,"method":"getblockcount","params":[]}
)
print(r.json()["result"])  # 12345
```

### JavaScript (fetch)

```javascript
fetch('http://localhost:18332/', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Basic ' + btoa('nfxuser:password')
    },
    body: JSON.stringify({
        jsonrpc: '2.0',
        id: 'test',
        method: 'getbalance',
        params: ['NFX1...']
    })
})
.then(res => res.json())
.then(data => console.log(data.result));
```

---

*Related: [C API](c.md) | [Go API](go.md) | [Examples](../examples/)*
