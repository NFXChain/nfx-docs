# C API Reference (nfx-bindings)

Complete C ABI reference for `libnfx.so`.

## Header

```c
#include <nfx_bindings.h>
```

## Core Types

### Error Codes

```c
typedef enum {
    NFX_OK = 0,
    NFX_ERR_INVALID_ARGUMENT = -1,
    NFX_ERR_NOT_INITIALIZED = -2,
    NFX_ERR_OUT_OF_MEMORY = -3,
    NFX_ERR_IO = -4,
    NFX_ERR_NETWORK = -5,
    NFX_ERR_NOT_FOUND = -6,
    NFX_ERR_INVALID_STATE = -7,
    NFX_ERR_CONSENSUS = -8,
    NFX_ERR_CRYPTO = -9,
    NFX_ERR_QUOTA_EXCEEDED = -10,
    NFX_ERR_BLOCKCHAIN = -11,
    NFX_ERR_AI = -12,
    NFX_ERR_UNKNOWN = -999
} nfx_error_t;
```

### Context (Opaque Handle)

```c
typedef struct nfx_context_t nfx_context_t;
```

All API calls require a context pointer.

## Lifecycle

```c
nfx_context_t* nfx_create(void);
```

Create new context. Returns `NULL` on OOM.

```c
nfx_error_t nfx_init(
    nfx_context_t* ctx,
    const char* config_path
);
```

Initialize with config file. Must be called before other functions.

```c
nfx_error_t nfx_init_default(nfx_context_t* ctx);
```

Initialize with built-in defaults (testnet mode).

```c
void nfx_destroy(nfx_context_t* ctx);
```

Free all resources. **Always call this.**

```c
const char* nfx_error_string(nfx_error_t err);
```

Returns human-readable error message. **Do not free** (static string).

---

## Node Control

```c
nfx_error_t nfx_node_start(nfx_context_t* ctx);
nfx_error_t nfx_node_stop(nfx_context_t* ctx);
bool nfx_node_is_running(nfx_context_t* ctx);
nfx_error_t nfx_node_wait(nfx_context_t* ctx);  // Blocks until shutdown
int nfx_node_get_pid(nfx_context_t* ctx);
```

---

## Blockchain Queries

```c
typedef struct {
    int64_t blocks;
    int64_t headers;
    int64_t best_block;
    double verification_progress;
    char chain[16];
} nfx_blockchain_info_t;

nfx_error_t nfx_get_blockchain_info(
    nfx_context_t* ctx,
    nfx_blockchain_info_t* out
);
```

```c
nfx_error_t nfx_get_block_count(
    nfx_context_t* ctx,
    int64_t* count
);
```

```c
nfx_error_t nfx_get_block(
    nfx_context_t* ctx,
    const char* hash_or_height,
    int verbose,  // 0=hex, 1=txids, 2=full
    char* block_out,
    size_t* block_out_len
);
```

Returns JSON-serialized block in `block_out`. Caller must free with `nfx_free()`.

---

## Transactions

```c
nfx_error_t nfx_get_balance(
    nfx_context_t* ctx,
    const char* address,
    uint64_t* balance
);
```

Balance in satoshis (smallest unit).

```c
typedef struct {
    char txid[65];
    uint32_t vout;
    uint64_t amount;
    char scriptPubKey[256];
    int64_t confirmations;
    bool spendable;
} nfx_utxo_t;

nfx_error_t nfx_list_unspent(
    nfx_context_t* ctx,
    const char* min_conf_str,  // e.g., "1"
    const char* max_conf_str,  // e.g., "9999999"
    nfx_utxo_t** utxos,
    size_t* count
);
```

**Memory:** Use `nfx_free(utxos)` when done.

```c
nfx_error_t nfx_send_transaction(
    nfx_context_t* ctx,
    const char* from,
    const char* to,
    uint64_t amount,
    uint64_t fee
);
```

Convenience wrapper. Returns TXID in `out_txid`.

```c
nfx_error_t nfx_broadcast_raw_tx(
    nfx_context_t* ctx,
    const uint8_t* tx_hex,
    size_t tx_len,
    char* txid_out,
    size_t* txid_out_len
);
```

Broadcast raw transaction hex.

---

## Mining & Consensus

```c
nfx_error_t nfx_get_mining_info(
    nfx_context_t* ctx,
    nfx_mining_info_t* out
);

typedef struct {
    int64_t blocks;
    double difficulty;
    char network[16];
    bool poes_enabled;
    double poes_security_factor;
} nfx_mining_info_t;
```

```c
nfx_error_t nfx_submit_block(
    nfx_context_t* ctx,
    const uint8_t* block_data,
    size_t block_len
);
```

---

## AI Governance

```c
nfx_error_t nfx_ai_get_status(
    nfx_context_t* ctx,
    bool* enabled,
    double* threshold_valid,
    double* threshold_suspicious
);

typedef struct {
    double valid;
    double suspicious;
    double malicious;
    char recommended_action[32];
    float confidence;
} nfx_ai_score_t;

nfx_error_t nfx_ai_score_block(
    nfx_context_t* ctx,
    const char* block_hash,
    nfx_ai_score_t* score
);
```

---

## Wallet Operations

```c
nfx_error_t nfx_wallet_create(
    nfx_context_t* ctx,
    const char* name,
    const char* passphrase,
    char* mnemonic_out,
    size_t* mnemonic_len
);
```

Creates new HD wallet. Returns 12-word mnemonic.

```c
nfx_error_t nfx_wallet_open(
    nfx_context_t* ctx,
    const char* uuid
);
```

```c
nfx_error_t nfx_wallet_get_address(
    nfx_context_t* ctx,
    int index,
    char* address_out,
    size_t* address_len
);
```

```c
nfx_error_t nfx_wallet_sign_message(
    nfx_context_t* ctx,
    const char* address,
    const char* message,
    char* signature_out,
    size_t* sig_len
);
```

---

## Network

```c
typedef struct {
    char address[46];
    uint64_t bytes_sent;
    uint64_t bytes_recv;
    double ping_time_ms;
    char version[50];
    bool inbound;
} nfx_peer_info_t;

nfx_error_t nfx_get_peer_info(
    nfx_context_t* ctx,
    nfx_peer_info_t** peers,
    size_t* count
);
```

**Free with:** `nfx_free(peers)`

---

## Memory Management

```c
void nfx_free(void* ptr);
```

Free memory allocated by library (e.g., from `nfx_list_unspent`, `nfx_get_block`).

---

## Thread Safety

- **Each thread** needs its own `nfx_context_t*`
- Do NOT share same context across threads without mutex
- Functions are thread-safe when using separate contexts

---

## Compiling C Programs

```bash
gcc -o myapp myapp.c -lnfx -L/path/to/lib
```

**Link dependencies:**
```bash
-lnfx -lleveldb -lssl -lQt5Core -lQt5Network -lpthread -ldl -lm
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include "nfx_bindings.h"

int main() {
    nfx_context_t* ctx = nfx_create();
    if (!ctx) { fprintf(stderr, "OOM\n"); return 1; }

    nfx_error_t err = nfx_init_default(ctx);
    if (err != NFX_OK) {
        fprintf(stderr, "Init error: %s\n", nfx_error_string(err));
        nfx_destroy(ctx);
        return 1;
    }

    // Start node
    err = nfx_node_start(ctx);
    if (err != NFX_OK) {
        fprintf(stderr, "Start failed: %s\n", nfx_error_string(err));
        nfx_destroy(ctx);
        return 1;
    }

    printf("Node running, PID: %d\n", nfx_node_get_pid(ctx));

    // Wait for shutdown
    nfx_node_wait(ctx);
    nfx_destroy(ctx);
    return 0;
}
```

---

*For more examples, see [docs/examples/](../examples/)*
