# Go API Reference

Complete API reference for the `nfx-go` package.

## Packages

- `github.com/NFXChain/nfx-go/rpc` â€” JSON-RPC client
- `github.com/NFXChain/nfx-go/rpc/ws` â€” WebSocket client
- `github.com/NFXChain/nfx-go/sdk/wallet` â€” Wallet management
- `github.com/NFXChain/nfx-go/sdk/txbuilder` â€” Transaction builder
- `github.com/NFXChain/nfx-go/sdk/hdwallet` â€” HD wallet (BIP-44)
- `github.com/NFXChain/nfx-go/sdk/encoder` â€” Address encoding

---

## RPC Client

### NewClient

```go
func NewClient(opts ...ClientOption) *Client
```

Creates a new RPC client.

**Options:**
```go
rpc.WithURL(url string)          // RPC endpoint (default: http://localhost:18332)
rpc.WithCredentials(user, pass)  // HTTP basic auth
rpc.WithTimeout(d time.Duration) // Request timeout (default: 30s)
rpc.WithRetry(n int)            // Retry failed requests (default: 0)
rpc.WithRetryDelay(d time.Duration)
rpc.WithHTTPClient(*http.Client) // Custom HTTP client
```

**Example:**
```go
client := rpc.NewClient(
    rpc.WithURL("http://localhost:18332"),
    rpc.WithCredentials("nfxuser", "password"),
    rpc.WithTimeout(10*time.Second),
)
```

### GetBlockchainInfo

```go
func (c *Client) GetBlockchainInfo() (*BlockchainInfo, error)
```

Returns an object containing various state info regarding blockchain processing.

**Response:**
```go
type BlockchainInfo struct {
    Chain                string  `json:"chain"`                // "main", "test", or "regtest"
    Blocks               int64   `json:"blocks"`               // Current height
    Headers              int64   `json:"headers"`              // Height of latest verified header
    BestBlockHash        string  `json:"bestblockhash"`        // Hash of best block
    Difficulty           float64 `json:"difficulty"`           // Current difficulty
    VerificationProgress float64 `json:"verificationprogress"` // 0.0-1.0
    SizeOnDisk           int64   `json:"size_on_disk"`
}
```

### GetBlock

```go
func (c *Client) GetBlock(hash string, verbose int) (*Block, error)
```

**Parameters:**
- `hash`: Block hash or height as string
- `verbose`:
  - `0` = hex string only
  - `1` = block with tx hashes
  - `2` = full block with all tx details

### GetBlockCount

```go
func (c *Client) GetBlockCount() (int64, error)
```

Current block height.

### GetBalance

```go
func (c *Client) GetBalance(address string) (uint64, error)
```

Balance in satoshis (or smallest unit).

### ListUnspent

```go
func (c *Client) ListUnspent(
    minConf, maxConf int,
    address ...string,
) ([]UTXO, error)
```

List unspent transaction outputs.

```go
type UTXO struct {
    TxID          string  `json:"txid"`
    Vout          uint32  `json:"vout"`
    Address       string  `json:"address"`
    Amount        float64 `json:"amount"`        // In NFX
    Confirmations int64   `json:"confirmations"`
    Spendable     bool    `json:"spendable"`
}
```

### SendTransaction

```go
func (c *Client) SendTransaction(req *SendTxRequest) (string, error)
```

```go
type SendTxRequest struct {
    From    string  `json:"from"`    // Sending address
    To      string  `json:"to"`      // Receiving address
    Amount  float64 `json:"amount"`  // Amount in NFX
    Fee     float64 `json:"fee"`     // Transaction fee in NFX
    Nonce   uint64  `json:"nonce"`   // Optional: override nonce
}
```

Returns transaction ID (TXID).

### GetTransaction

```go
func (c *Client) GetTransaction(txid string) (*Transaction, error)
```

```go
type Transaction struct {
    TXID          string    `json:"txid"`
    Hash          string    `json:"hash"`
    Version       int32     `json:"version"`
    Vin           []TxIn    `json:"vin"`
    Vout          []TxOut   `json:"vout"`
    LockTime      uint32    `json:"locktime"`
    BlockHash     string    `json:"blockhash,omitempty"`
    BlockHeight   int64     `json:"blockheight,omitempty"`
    Confirmations int64     `json:"confirmations,omitempty"`
    Time          int64     `json:"time"`
}
```

### GetMempoolInfo

```go
func (c *Client) GetMempoolInfo() (*MempoolInfo, error)
```

```go
type MempoolInfo struct {
    Size      int64   `json:"size"`       // TX count
    Bytes     int64   `json:"bytes"`      // Total size
    Usage      int64   `json:"usage"`      // Memory usage
}
```

### GetPeerInfo

```go
func (c *Client) GetPeerInfo() ([]PeerInfo, error)
```

### GetNetworkInfo

```go
func (c *Client) GetNetworkInfo() (*NetworkInfo, error)
```

---

## WebSocket Client

### NewWebSocketClient

```go
func NewClient(url string, opts ...WSOption) (*WSClient, error)
```

### Subscribe

```go
func (c *WSClient) Subscribe(method string, handler func(Message))
```

Subscribe to event stream.

**Methods:**
- `"block"` â€” New block notifications
- `"tx"` â€” Confirmed transactions
- `"rawtx"` â€” Mempool transactions
- `"address.<addr>"` â€” Transactions for address
- `"alert"` â€” Network alerts
- `"ai_score"` â€” AI governance results

**Example:**
```go
ws, _ := ws.NewClient("ws://localhost:18334")
ws.Subscribe("block", func(msg ws.Message) {
    fmt.Println("New block:", msg.Data["hash"])
})
ws.Listen() // Blocks
```

### Unsubscribe

```go
func (c *WSClient) Unsubscribe(method string)
```

---

## Wallet SDK

### NewManager

```go
func NewManager(rpc *rpc.Client, opts ...WalletOption) *WalletManager
```

### CreateWallet

```go
func (m *WalletManager) CreateWallet(name, passphrase string) (string, error)
```

Returns mnemonic (12-word seed). **Save this securely!**

### GetAddress

```go
func (m *WalletManager) GetAddress(index int) (string, error)
```

Returns address at derivation path `m/44'/777'/0'/0/{index}`.

### GetBalance

```go
func (m *WalletManager) GetBalance(addr string) (uint64, error)
```

Wallet's cached balance (auto-refreshes).

### Send

```go
func (m *WalletManager) Send(
    from string,
    to string,
    amount float64,
    fee float64,
) (string, error)
```

Broadcasts transaction. Returns TXID.

### ListTransactions

```go
func (m *WalletManager) ListTransactions(
    addr string,
    since time.Time,
) ([]Transaction, error)
```

---

## Transaction Builder

### NewBuilder

```go
func NewBuilder(rpc *rpc.Client) *Builder
```

### From

```go
func (b *Builder) From(addr string) *Builder
```

Set sender address.

### AddOutput

```go
func (b *Builder) AddOutput(addr string, amount float64) *Builder
```

Add recipient.

### SetFee

```go
func (b *Builder) SetFee(amount float64) *Builder
```

Fee in NFX.

### SetChange

```go
func (b *Builder) SetChange(addr string) *Builder
```

Change address (defaults to sender).

### Build

```go
func (b *Builder) Build() ([]byte, error)
```

Returns raw signed transaction hex.

**Example:**
```go
tx, err := txbuilder.New(client).
    From("NFX1sender...").
    AddOutput("NFX1receiver...", 1.5).
    SetFee(0.001).
    SetChange("NFX1sender...").
    Build()
```

---

## HD Wallet

### GenerateMnemonic

```go
func GenerateMnemonic(strength int) (string, error)
```

**Strength:** 128 (12 words), 160 (15 words), 192 (18 words), 224 (21 words), 256 (24 words)

### FromMnemonic

```go
func FromMnemonic(mnemonic string) (*HDWallet, error)
```

### Derive

```go
func (w *HDWallet) Derive(path string) (*ExtendedKey, error)
```

Standard BIP-44 path for NFX Chain: `m/44'/777'/0'/0/{index}`

**Example:**
```go
wallet, _ := hdwallet.FromMnemonic("abandon ...")
key, _ := wallet.Derive("m/44'/777'/0'/0/0")
addr := key.Address()  // NFX1...
wif := key.WIF()       // L1aW4a...
```

---

## Types & Constants

### Common

```go
// Address validation
func IsValidAddress(addr string) bool

// Base58Check decode
func DecodeBase58(addr string) ([]byte, error)

// Encode to address
func AddressFromPubKey(pubkey []byte) (string, error)
```

---

*Full auto-generated API docs: `go doc github.com/NFXChain/nfx-go/rpc`*
