# nfx-bindings Module Documentation

**Module:** `nfx-bindings`  
**Language:** C++ (C ABI)  
**Type:** Shared Library (.so/.dylib)  
**Path:** `nfx-bindings/`  

## Overview

`nfx-bindings` provides a C-compatible Foreign Function Interface (FFI) to `nfx-core`. It exposes core functionality through a clean C API that can be called from:

- **Go** (via CGo)
- **Python** (via ctypes or cffi)
- **Rust** (via libloading or bindgen)
- **Node.js** (via node-ffi-napi)
- **Any language with C FFI support**

## Architecture

```
+â”€â”€â”€â”€â”€â”€â”€â”€â”€+     +â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€+     +â”€â”€â”€â”€â”€â”€â”€â”€â”€+
â”‚   Go     â”‚â”€â”€â”€â”€â–ºâ”‚  C ABI     â”‚â”€â”€â”€â”€â–ºâ”‚ nfx-    â”‚
â”‚  (CGo)   â”‚     â”‚  Bindings  â”‚     â”‚ core    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€+â”˜     +â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€+â”˜     +â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                       â”‚
                 libnfx.so
```

The bindings layer acts as a translation layer:
- Converts C types to C++ objects
- Manages memory allocation/deallocation
- Provides thread-safe wrappers
- Handles error conversion

## Directory Structure

```
nfx-bindings/
â”œâ”€â”€ CMakeLists.txt
â”œâ”€â”€ README.md
â”œâ”€â”€ cgo_bindings.cpp     # Main C API implementation
â”œâ”€â”€ cgo_bindings.hpp     # C API declarations (extern "C")
â””â”€â”€ include/             # Optional: additional headers
```

## Building

```bash
cd nfx-bindings

# Create build directory
mkdir build && cd build

# Configure (requires nfx-core)
cmake .. \
    -DNFX_CORE_INCLUDE_DIR=../nfx-core/include \
    -DNFX_CORE_LIBRARY=../nfx-core/build/libnfx-core.so

# Or if nfx-core installed system-wide
# cmake ..

# Build shared library
make -j$(nproc)

# Output: libnfx.so (Linux), libnfx.dylib (macOS), nfx.dll (Windows)
```

## API Reference

All functions are prefixed with `nfx_` to avoid namespace collisions.

### Context Management

```c
// Opaque context handle
typedef struct nfx_context_t nfx_context_t;

// Create new context
nfx_context_t* nfx_create(void);

// Initialize context with config file
nfx_error_t nfx_init(
    nfx_context_t* ctx,
    const char* config_path
);

// Initialize with default config
nfx_error_t nfx_init_default(nfx_context_t* ctx);

// Destroy context and free all resources
void nfx_destroy(nfx_context_t* ctx);

// Get last error string
const char* nfx_error_string(nfx_error_t err);
```

**Usage Example (C):**

```c
#include <stdio.h>
#include "nfx_bindings.h"

int main() {
    nfx_context_t* ctx = nfx_create();
    nfx_error_t err = nfx_init(ctx, "config.toml");

    if (err != NFX_OK) {
        printf("Failed: %s\n", nfx_error_string(err));
        nfx_destroy(ctx);
        return 1;
    }

    // ... use context ...

    nfx_destroy(ctx);
    return 0;
}
```

### Error Handling

All API functions return `nfx_error_t`:

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

### Node Lifecycle

```c
// Start node (begins P2P and RPC)
nfx_error_t nfx_node_start(nfx_context_t* ctx);

// Stop node gracefully
nfx_error_t nfx_node_stop(nfx_context_t* ctx);

// Check if node is running
bool nfx_node_is_running(nfx_context_t* ctx);

// Wait for shutdown (blocking)
nfx_error_t nfx_node_wait(nfx_context_t* ctx);
```

### Blockchain Queries

```c
// Get blockchain info
typedef struct {
    int64_t blocks;
    int64_t headers;
    int64_t best_block;
    double verification_progress;
    char chain[16];
} nfx_blockchain_info_t;

nfx_error_t nfx_get_blockchain_info(
    nfx_context_t* ctx,
    nfx_blockchain_info_t* info
);

// Get block by hash or height
nfx_error_t nfx_get_block(
    nfx_context_t* ctx,
    const char* hash_or_height,
    bool verbose,
    nfx_block_t* block
);

// Get block count
nfx_error_t nfx_get_block_count(
    nfx_context_t* ctx,
    int64_t* count
);

// Get best block hash
nfx_error_t nfx_get_best_block_hash(
    nfx_context_t* ctx,
    char* hash_out,
    size_t hash_out_len
);
```

### Transactions

```c
// Get balance for address
nfx_error_t nfx_get_balance(
    nfx_context_t* ctx,
    const char* address,
    uint64_t* balance
);

// Get unspent outputs (UTXOs)
typedef struct {
    char txid[65];      // Hex string
    uint32_t vout;
    uint64_t amount;
    char scriptPubKey[256];
    bool confirmed;
} nfx_utxo_t;

nfx_error_t nfx_list_unspent(
    nfx_context_t* ctx,
    const char* min_conf,
    const char* max_conf,
    nfx_utxo_t** utxos,
    size_t* count
);

// Create raw transaction
nfx_error_t nfx_create_tx(
    nfx_context_t* ctx,
    const nfx_tx_input_t* inputs,
    size_t inputs_count,
    const nfx_tx_output_t* outputs,
    size_t outputs_count,
    char* raw_tx_hex,
    size_t* raw_tx_len
);

// Send transaction
nfx_error_t nfx_send_transaction(
    nfx_context_t* ctx,
    const char* raw_tx_hex
);

// Broadcast raw transaction
nfx_error_t nfx_broadcast_tx(
    nfx_context_t* ctx,
    const uint8_t* tx_data,
    size_t tx_len
);
```

**Transaction Structures:**

```c
// Transaction input
typedef struct {
    char txid[65];       // Previous transaction hash
    uint32_t vout;       // Output index
    char script_sig[256]; // Unlocking script
    uint32_t sequence;   // Sequence number
} nfx_tx_input_t;

// Transaction output
typedef struct {
    char address[100];   // Destination address (P2PKH/P2SH)
    uint64_t amount;     // Amount in satoshis
    char script[256];    // Locking script (optional if address provided)
} nfx_tx_output_t;
```

### Mining & Consensus

```c
// Get mining info
typedef struct {
    int64_t blocks;
    int64_t difficulty;
    char network[16];
    bool poes_enabled;
    double poes_security_factor;
} nfx_mining_info_t;

nfx_error_t nfx_get_mining_info(
    nfx_context_t* ctx,
    nfx_mining_info_t* info
);

// Submit block (for miners)
nfx_error_t nfx_submit_block(
    nfx_context_t* ctx,
    const uint8_t* block_data,
    size_t block_len
);

// Get work for mining (if solo mining)
nfx_error_t nfx_get_work(
    nfx_context_t* ctx,
    char* work_data,
    size_t* work_len
);
```

### AI Governance

```c
// Get AI validation status
nfx_error_t nfx_ai_get_status(
    nfx_context_t* ctx,
    bool* enabled,
    double* threshold_valid,
    double* threshold_suspicious
);

// Get AI score for a block
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

// Disable AI (admin only)
nfx_error_t nfx_ai_set_enabled(
    nfx_context_t* ctx,
    bool enabled
);
```

### Network Information

```c
// Get peer info
typedef struct {
    char address[46];       // IP:port
    char services[20];      // Service bits hex
    int64_t bytes_sent;
    int64_t bytes_recv;
    double ping_time;
    char version[50];
    bool inbound;
} nfx_peer_info_t;

nfx_error_t nfx_get_peer_info(
    nfx_context_t* ctx,
    nfx_peer_info_t** peers,
    size_t* count
);

// Get network statistics
nfx_error_t nfx_get_network_stats(
    nfx_context_t* ctx,
    nfx_network_stats_t* stats
);
```

## Using from Go (CGo)

Go can directly call C functions via CGo. Example:

### Basic Usage

```go
package main

/*
#include "nfx_bindings.h"
*/
import "C"
import (
    "fmt"
    "unsafe"
)

func main() {
    ctx := C.nfx_create()
    defer C.nfx_destroy(ctx)

    config := C.CString("config.toml")
    defer C.free(unsafe.Pointer(config))

    err := C.nfx_init(ctx, config)
    if err != C.NFX_OK {
        msg := C.nfx_error_string(err)
        fmt.Printf("Init failed: %s\n", C.GoString(msg))
        return
    }

    // Start node (async)
    go func() {
        err = C.nfx_node_start(ctx)
        if err != C.NFX_OK {
            fmt.Println("Node start error:", err)
        }
    }()

    // Query balance
    addr := C.CString("NFX123...")
    defer C.free(unsafe.Pointer(addr))

    var balance C.uint64_t
    err = C.nfx_get_balance(ctx, addr, &balance)
    if err == C.NFX_OK {
        fmt.Printf("Balance: %d\n", balance)
    }

    // Keep running
    select {}
}
```

### Wrapper Library

For better Go ergonomics, create a wrapper:

```go
// nfx.go
package nfx

/*
#cgo LDFLAGS: -L. -lnfx
#include "nfx_bindings.h"
*/
import "C"
import (
    "errors"
    "unsafe"
)

type Context struct {
    ctx *C.nfx_context_t
}

func New(configPath string) (*Context, error) {
    ctx := C.nfx_create()
    cpath := C.CString(configPath)
    defer C.free(unsafe.Pointer(cpath))

    if err := C.nfx_init(ctx, cpath); err != C.NFX_OK {
        return nil, errors.New(C.GoString(C.nfx_error_string(err)))
    }
    return &Context{ctx: ctx}, nil
}

func (c *Context) Start() error {
    if err := C.nfx_node_start(c.ctx); err != C.NFX_OK {
        return errors.New(C.GoString(C.nfx_error_string(err)))
    }
    return nil
}

func (c *Context) GetBalance(address string) (uint64, error) {
    caddr := C.CString(address)
    defer C.free(unsafe.Pointer(caddr))

    var balance C.uint64_t
    if err := C.nfx_get_balance(c.ctx, caddr, &balance); err != C.NFX_OK {
        return 0, errors.New(C.GoString(C.nfx_error_string(err)))
    }
    return uint64(balance), nil
}

func (c *Context) Close() {
    C.nfx_destroy(c.ctx)
}

// More methods...
```

**Usage:**

```go
package main

import (
    "log"
    "github.com/yourusername/nfx-go/nfx"
)

func main() {
    client, err := nfx.New("config.toml")
    if err != nil {
        log.Fatal(err)
    }
    defer client.Close()

    if err := client.Start(); err != nil {
        log.Fatal(err)
    }

    balance, _ := client.GetBalance("NFX123...")
    log.Printf("Balance: %d\n", balance)

    select {} // Keep alive
}
```

## Using from Python (ctypes)

```python
import ctypes
from ctypes import c_void_p, c_char_p, c_uint64, c_int, c_bool
import json

# Load library
lib = ctypes.CDLL('./libnfx.so')

# Define types
nfx_context_p = c_void_p
nfx_error_t = c_int
nfx_bool = c_bool

# Context functions
lib.nfx_create.restype = nfx_context_p
lib.nfx_destroy.argtypes = [nfx_context_p]
lib.nfx_init.argtypes = [nfx_context_p, c_char_p]
lib.nfx_init.restype = nfx_error_t

class NFXClient:
    def __init__(self, config_path):
        self.ctx = lib.nfx_create()
        err = lib.nfx_init(self.ctx, config_path.encode())
        if err != 0:
            raise Exception(f"Init failed: {err}")

    def get_balance(self, address):
        class BalanceResult:
            pass

        result = BalanceResult()
        lib.nfx_get_balance.argtypes = [
            nfx_context_p, c_char_p,
            ctypes.POINTER(c_uint64)
        ]
        lib.nfx_get_balance.restype = nfx_error_t

        bal = c_uint64(0)
        err = lib.nfx_get_balance(
            self.ctx,
            address.encode(),
            ctypes.byref(bal)
        )
        if err != 0:
            raise Exception(f"Get balance failed: {err}")
        return bal.value

    def close(self):
        lib.nfx_destroy(self.ctx)

# Usage
client = NFXClient("config.toml")
print(client.get_balance("NFX123..."))
client.close()
```

## Using from Rust

```rust
use std::ffi::{CString, CStr};
use std::os::raw::{c_char, c_void, c_int, c_uint64};

#[repr(C)]
pub struct NfxContext {
    _private: [u8; 0],
}

extern "C" {
    fn nfx_create() -> *mut NfxContext;
    fn nfx_destroy(ctx: *mut NfxContext);
    fn nfx_init(ctx: *mut NfxContext, config: *const c_char) -> i32;
    fn nfx_get_balance(
        ctx: *mut NfxContext,
        address: *const c_char,
        balance: *mut c_uint64
    ) -> i32;
}

pub struct NfxClient {
    ctx: *mut NfxContext,
}

impl NfxClient {
    pub fn new(config_path: &str) -> Result<Self, String> {
        let ctx = unsafe { nfx_create() };
        let cpath = CString::new(config_path).unwrap();

        let err = unsafe { nfx_init(ctx, cpath.as_ptr()) };
        if err != 0 {
            unsafe { nfx_destroy(ctx) };
            Err(format!("Init failed: {}", err))
        } else {
            Ok(Self { ctx })
        }
    }

    pub fn get_balance(&self, address: &str) -> Result<u64, String> {
        let caddr = CString::new(address).unwrap();
        let mut balance: u64 = 0;

        let err = unsafe {
            nfx_get_balance(
                self.ctx,
                caddr.as_ptr(),
                &mut balance as *mut u64
            )
        };

        if err == 0 {
            Ok(balance)
        } else {
            Err(format!("Get balance failed: {}", err))
        }
    }
}

impl Drop for NfxClient {
    fn drop(&mut self) {
        unsafe { nfx_destroy(self.ctx) };
    }
}
```

## Memory Management Rules

### Ownership conventions:

| Function | Memory Ownership |
|----------|-----------------|
| `nfx_create()` | Caller receives pointer, must call `nfx_destroy()` |
| `nfx_error_string()` | Returns internal static string, do not free |
| `nfx_list_unspent()` | Returns allocated array, caller must `free()` with `nfx_free()` |
| String params | Caller allocates, callee copies |
| String outputs | Callee allocates, caller frees with `nfx_free_string()` |

**Example:**

```c
nfx_utxo_t* utxos = NULL;
size_t count = 0;

nfx_error_t err = nfx_list_unspent(ctx, "1", "999999", &utxos, &count);
if (err == NFX_OK) {
    for (size_t i = 0; i < count; i++) {
        printf("TXID: %s\n", utxos[i].txid);
    }
    nfx_free(utxos);  // IMPORTANT: free allocated array
}
```

## Thread Safety

The bindings layer is **thread-safe** when using separate contexts:

```c
// Thread 1
nfx_context_t* ctx1 = nfx_create();
nfx_init(ctx1, "config1.toml");
nfx_node_start(ctx1);

// Thread 2 (separate context, OK)
nfx_context_t* ctx2 = nfx_create();
nfx_init(ctx2, "config2.toml");
nfx_node_start(ctx2);  // Both nodes run independently

// DON'T share same context across threads without mutex:
// nfx_context_t* shared_ctx = ...;
// Using from multiple threads = UNDEFINED BEHAVIOR
```

**Recommendation:** One context per thread, or use mutex to protect shared context.

## Advanced: Custom Memory Allocator

```c
// Set custom allocator (optional)
typedef void* (*nfx_alloc_func)(size_t size);
typedef void (*nfx_free_func)(void* ptr);

nfx_error_t nfx_set_allocator(
    nfx_alloc_func alloc,
    nfx_free_func free
);

// Example: use jemalloc or tcmalloc for better performance
```

## Packaging

When packaging for distribution:

### Linux
```bash
# Install library
sudo cp libnfx.so /usr/local/lib/
sudo cp nfx_bindings.h /usr/local/include/
sudo ldconfig

# Or use pkg-config
sudo cp nfx-bindings.pc /usr/local/lib/pkgconfig/
```

### macOS
```bash
# Install with Homebrew formula
brew install nfx-bindings

# Or manually
cp libnfx.dylib /usr/local/lib/
cp nfx_bindings.h /usr/local/include/
```

### Windows (WSL2/Linux Subsystem)
```bash
# Not officially supported on native Windows
# Use WSL2 or compile with MinGW (community)
# See: https://github.com/NFXChain/nfx-bindings/issues/42
```

## Troubleshooting Bindings

### Error: `undefined reference to nfx_*`

**Cause:** Linker can't find `libnfx.so` or order is wrong.

**Solution:**
```bash
# Ensure library path
export LD_LIBRARY_PATH=/path/to/nfx-core/build:$LD_LIBRARY_PATH

# Link in correct order (dependencies after)
gcc -o myapp myapp.c -L. -lnfx -lleveldb -lssl -lQt5Core -lQt5Network
```

### Error: `cannot open shared object file: No such file or directory`

**Fix:**
```bash
# Copy library to standard location
sudo cp libnfx.so /usr/local/lib/
sudo ldconfig

# Or run with rpath
gcc -o myapp myapp.c -Wl,-rpath,/path/to/lib -L. -lnfx
```

### Go CGO errors

**Error:** `cgo: C compiler not found`

**Fix:**
```bash
# Install gcc
sudo apt install gcc

# Ensure CGO_ENABLED=1
export CGO_ENABLED=1
go build
```

### Python ctypes OSError

**Error:** `OSError: libnfx.so: cannot open shared object file`

**Fix:**
```python
import ctypes
# Add library path
ctypes.cdll.LoadLibrary('/full/path/to/libnfx.so')
# Or set LD_LIBRARY_PATH in shell
```

## Performance Tips

### 1. Batch Operations

Instead of:
```c
for (int i = 0; i < 1000; i++) {
    nfx_get_balance(ctx, addr, &balance);  // SLOW: 1000 calls
}
```

Use batch API (if available) or implement yourself:

```c
// Future: nfx_get_balances_multi()
nfx_balance_t* balances;
size_t count;
nfx_get_balances(ctx, addresses, &balances, &count);
```

### 2. Reuse Context

```c
// Good: reuse context
nfx_context_t* ctx = nfx_create();
nfx_init(ctx, "config.toml");
for (int i = 0; i < 10000; i++) {
    nfx_get_balance(ctx, addr, &balance);  // Fast
}
nfx_destroy(ctx);

// Bad: create/destroy every call
for (int i = 0; i < 10000; i++) {
    ctx = nfx_create();
    nfx_init(ctx, "config.toml");
    nfx_get_balance(ctx, addr, &balance);
    nfx_destroy(ctx);  // Very slow!
}
```

### 3. Async Operations (Go)

```go
// Use goroutines for parallel calls
var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(i int) {
        defer wg.Done()
        balance, _ := client.GetBalance(addr)
        fmt.Println(balance)
    }(i)
}
wg.Wait()
```

## Testing the Bindings

### Unit Tests (C)

```c
#include "test_runner.h"

TEST(test_get_balance) {
    nfx_context_t* ctx = nfx_create();
    nfx_init(ctx, "test_config.toml");

    uint64_t balance;
    nfx_error_t err = nfx_get_balance(ctx, "NFX123...", &balance);

    ASSERT_EQ(err, NFX_OK);
    ASSERT_GT(balance, 0);

    nfx_destroy(ctx);
}

int main() {
    TestRunner_run();
    return 0;
}
```

Build and run:
```bash
gcc -o test_bindings test_bindings.c -lnfx -L. && ./test_bindings
```

### Fuzzing

Use libFuzzer for C API fuzzing:

```cpp
extern "C" int LLVMFuzzerTestOneInput(const uint8_t* data, size_t size) {
    nfx_context_t* ctx = nfx_create();
    nfx_init_default(ctx);

    // Feed fuzzed data as RPC request
    char* raw = strdup((const char*)data);
    nfx_handle_raw_message(ctx, raw, size);
    free(raw);

    nfx_destroy(ctx);
    return 0;
}
```

## Contributing

When adding new C API:

1. Declare in `cgo_bindings.hpp` with `extern "C"`
2. Implement in `cgo_bindings.cpp`
3. Add to `CMakeLists.txt` exports
4. Write C test case in `tests/`
5. Add Go wrapper example in `examples/go/`
6. Document in `docs/api/c.md`

---

*Next: [Go API Module](go.md) | [Wallet Module](wallet.md) | [API Reference](../api/)*
