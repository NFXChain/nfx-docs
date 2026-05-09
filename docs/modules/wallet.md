# nfx-wallet Module Documentation

**Module:** `nfx-wallet`  
**Language:** C++17, Qt5 (Widgets & QML), JavaScript (QML)  
**Type:** Desktop GUI Application  
**Path:** `nfx-wallet/`  

## Overview

`nfx-wallet` is the official cross-platform desktop wallet for NFX Chain. It provides a modern, user-friendly interface for:

- Managing UTXO-based wallet (create/import/backup)
- Sending and receiving NFX tokens
- Staking participation (PoS/PoES)
- Monitoring hypernodes and network status
- Interacting with smart contracts
- Transaction history with filters
- QR code generation/scanning

## Screenshots

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  NFX Wallet                    [â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘] â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  Overview           â”‚  Send    â”‚  Receive   â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                     â”‚          â”‚            â”‚
â”‚  Balance: 1,250 NFX â”‚          â”‚            â”‚
â”‚  24h Change: +2.3%  â”‚          â”‚            â”‚
â”‚                     â”‚          â”‚            â”‚
â”‚  Recent Transactionsâ”‚          â”‚            â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜          â”‚            â”‚
â”‚  â€¢ +1.5 NFX from ...           â”‚            â”‚
â”‚  â€¢ -0.5 NFX to ...             â”‚  QR Code   â”‚
â”‚  â€¢ +10 NFX staking reward      â”‚  Address   â”‚
â”‚                                â”‚            â”‚
â”‚  Staking: 500 NFX              â”‚  Copy      â”‚
â”‚  â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘â–‘â–‘ 40%              â”‚            â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

## Directory Structure

```
nfx-wallet/
â”œâ”€â”€ CMakeLists.txt
â”œâ”€â”€ README.md
â”œâ”€â”€ nfx_wallet.ui              # Qt Designer form (main window)
â”œâ”€â”€ nfx_wallet.cpp/h           # Main application class
â”œâ”€â”€ main.cpp                   # Entry point
â”œâ”€â”€ main_qml.cpp               # QML-only variant
â”œâ”€â”€ main_widget.cpp            # Widget-based variant
â”‚
â”œâ”€â”€ src/
â”‚   â”œâ”€â”€ nfxapi.cpp/h          # RPC client implementation
â”‚   â”œâ”€â”€ walletdatabase.cpp/h  # SQLite wallet DB
â”‚   â”œâ”€â”€ walletmanager.cpp/h   # Wallet logic & key management
â”‚   â”œâ”€â”€ nodediscovery.cpp/h   # Node auto-detection
â”‚   â”œâ”€â”€ logger.cpp/h          # Logging subsystem
â”‚   â”‚
â”‚   â”œâ”€â”€ nfx/                  # NFX-specific models
â”‚   â”‚   â”œâ”€â”€ nfxmodel.cpp/h    # Data models for list views
â”‚   â”‚   â””â”€â”€ ...               â”‚
â”‚   â”‚
â”‚   â”œâ”€â”€ rpc/                  # RPC wrappers
â”‚   â”‚   â”œâ”€â”€ rpcclient.cpp/h
â”‚   â”‚   â””â”€â”€ ...               â”‚
â”‚   â”‚
â”‚   â”œâ”€â”€ wallet/               # Wallet operations
â”‚   â”‚   â”œâ”€â”€ wallet.cpp/h      # Core wallet logic
â”‚   â”‚   â”œâ”€â”€ walletmodel.cpp/h # Wallet model for UI
â”‚   â”‚   â””â”€â”€ ...               â”‚
â”‚   â”‚
â”‚   â””â”€â”€ widgets/              # Custom Qt widgets
â”‚       â”œâ”€â”€ receiverequestdialog.cpp/h
â”‚       â”œâ”€â”€ sendcoinsdialog.cpp/h
â”‚       â””â”€â”€ transactiontablemodel.cpp/h
â”‚
â”œâ”€â”€ forms/                     # Qt Designer .ui files
â”‚   â”œâ”€â”€ mainwindow.ui
â”‚   â”œâ”€â”€ sendcoinsdialog.ui
â”‚   â””â”€â”€ receiverequestdialog.ui
â”‚
â”œâ”€â”€ translations/
â”‚   â”œâ”€â”€ nfx-qt_pt_BR.ts      # Brazilian Portuguese translation
â”‚   â””â”€â”€ ...                  # Other languages
â”‚
â”œâ”€â”€ icons/                    # Application icons
â”‚   â”œâ”€â”€ nfx_icon.png
â”‚   â”œâ”€â”€ nfx_icon.ico
â”‚   â”œâ”€â”€ about.png
â”‚   â””â”€â”€ ...                  # Toolbar icons
â”‚
â”œâ”€â”€ resources/
â”‚   â”œâ”€â”€ resources.qrc        # Qt resource file
â”‚   â””â”€â”€ icons/               # Embedded icons
â”‚
â””â”€â”€ qml/                      # Optional QML-only UI
    â”œâ”€â”€ MainView.qml
    â”œâ”€â”€ OverviewPage.qml
    â”œâ”€â”€ SendPage.qml
    â””â”€â”€ ...
```

## Building

### Dependencies

```bash
# Ubuntu/Debian
sudo apt install \
    qtbase5-dev \
    qt5-qmake \
    qtbase5-dev-tools \
    qtdeclarative5-dev \
    qml-module-qtquick2 \
    qml-module-qtquick-controls2 \
    qml-module-qtquick-dialogs \
    qml-module-qtquick-layouts \
    qml-module-qtgraphicaleffects \
    libsqlite3-dev
```

### Compile

```bash
cd nfx-wallet
mkdir build && cd build
qmake ..  # or cmake if using CMakeLists.txt (Qt6 style)
make -j$(nproc)
```

**Output:** `nfx-wallet` binary

## Running

```bash
# From build directory
./nfx-wallet

# From project root (if installed)
nfx-wallet --testnet

# With custom config
nfx-wallet --config ~/.nfx/config.toml

# Debug mode
nfx-wallet --log-level=debug --console
```

### Command-Line Options

```
Usage: nfx-wallet [OPTIONS]

Options:
  --testnet              Use testnet network
  --mainnet              Use mainnet network
  --config PATH          Path to config.toml
  --datadir PATH         Data directory (~/.nfx)
  --walletdir PATH       Wallet directory (~/.nfx/wallets)
  --create-wallet NAME   Create new wallet on startup
  --load-wallet NAME     Load specific wallet
  --daemon               Run in background (without UI)
  --no-gui               Headless mode (CLI only)
  --reset                Reset wallet (delete all data) - USE WITH CAUTION
  --help, -h             Show help
  --version              Show version
```

## UI Architecture

### Technology Stack

- **Framework:** Qt5 (C++ backend) + Qt Quick (QML frontend)
- **Layout:** Hybrid (main window in Widgets, pages in QML)
- **Styling:** QML-based theme, CSS for widgets
- **Model-View:** QAbstractItemModel for transaction lists

### Main Window (Widget-based)

**File:** `nfx_wallet.cpp` / `nfx_wallet.ui`

```cpp
class NFXWallet : public QMainWindow {
    Q_OBJECT

public:
    NFXWallet(QWidget* parent = nullptr);
    ~NFXWallet();

private slots:
    void onOverviewClicked();
    void onSendClicked();
    void onReceiveClicked();
    void onStakingClicked();
    void onGuardiansClicked();

private:
    void setupTabs();
    void loadSettings();
    void saveSettings();

    Ui::NFXWallet ui;           // Generated from .ui file

    WalletManager* walletMgr;   // Core wallet logic
    NFXAPI* rpcClient;          // RPC communication
    NodeDiscovery* nodeDisc;    // Node finder

    QSystemTrayIcon* trayIcon;  // System tray
    QMenu* trayMenu;
};
```

### QML Pages

Modern UI uses QML for fluid animations:

**MainView.qml:**
```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import "pages"

ApplicationWindow {
    visible: true
    width: 1000
    height: 700
    title: "NFX Wallet"

    // Navigation drawer
    Drawer {
        id: drawer
        edge: Qt.LeftEdge
        width: 250

        Column {
            spacing: 10
            padding: 20

            Label { text: "NFX Wallet"; font.bold: true; font.pixelSize: 20 }

            Button { text: "Overview"; onClicked: stackView.push("OverviewPage.qml") }
            Button { text: "Send"; onClicked: stackView.push("SendPage.qml") }
            Button { text: "Receive"; onClicked: stackView.push("ReceivePage.qml") }
            Button { text: "Staking"; onClicked: stackView.push("StakingPage.qml") }
            Button { text: "Guardians"; onClicked: stackView.push("GuardiansPage.qml") }
            Button { text: "Settings"; onClicked: stackView.push("SettingsPage.qml") }
        }
    }

    StackView {
        id: stackView
        anchors.fill: parent
        initialItem: "OverviewPage.qml"
    }
}
```

### Pages

| Page | File | Description |
|------|------|-------------|
| Overview | `OverviewPage.qml` | Balance, recent transactions, staking stats |
| Send | `SendPage.qml` | Send form with address validator |
| Receive | `ReceivePage.qml` | QR code display, address copy |
| Staking | `StakingPage.qml` | Stake management and rewards |
| Guardians | `GuardiansPage.qml` | Hypernode monitoring |

## Core Components

### 1. WalletManager

**File:** `src/walletmanager.cpp`

Central class handling wallet operations:

```cpp
class WalletManager : public QObject {
    Q_OBJECT

public:
    explicit WalletManager(QObject* parent = nullptr);
    ~WalletManager();

    // Wallet operations
    QString CreateWallet(
        const QString& name,
        const QString& passphrase
    );  // Returns mnemonic

    bool OpenWallet(const QString& uuid);
    void CloseWallet();

    // Encryption
    bool ChangePassphrase(const QString& old_pass, const QString& new_pass);
    bool EncryptWallet(const QString& passphrase);
    bool IsEncrypted() const;

    // Keys & addresses
    QString GetAddress(int index = 0) const;
    QByteArray GetPrivateKey(int index, const QString& passphrase);
    QString GetPublicKey(int index) const;

    // Balance & UTXOs
    uint64_t GetBalance(const QString& address) const;
    QList<UTXO> ListUnspent(const QString& address = QString());
    void UpdateBalance();  // Refresh from node

    // Transactions
    QString Send(
        const QString& from,
        const QString& to,
        double amount,
        double fee = 0.001
    );

    QList<Transaction> GetHistory(
        const QString& address = QString(),
        int confirmations = 0
    );

    // Staking
    bool Stake(const QString& address, uint64_t amount);
    bool Unstake(const QString& address, uint64_t amount);
    uint64_t GetStakedBalance() const;

    // Signals (for UI updates)
    Q_SIGNAL void balanceChanged(uint64_t new_balance);
    Q_SIGNAL void transactionReceived(const QString& txid);
    Q_SIGNAL void stakingUpdated();

private:
    WalletDatabase* db_;           // SQLite handler
    NFXAPI* rpc_;                  // RPC client
    QMap<QString, Key> keys_;      // Decrypted keys (in memory only)
    QTimer* refreshTimer_;         // Balance polling
};
```

**Wallet Database (SQLite Schema):**

```sql
-- wallets table (encrypted metadata)
CREATE TABLE wallets (
    uuid TEXT PRIMARY KEY,        -- Wallet UUID
    name TEXT NOT NULL,
    mnemonic_enc BLOB,            -- Encrypted mnemonic
    salt BLOB NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- keys table (derived keys, encrypted)
CREATE TABLE keys (
    id INTEGER PRIMARY KEY,
    wallet_uuid TEXT,
    address TEXT UNIQUE,
    pubkey BLOB,
    privkey_enc BLOB,             -- Encrypted private key
    derivation_path TEXT,
    FOREIGN KEY(wallet_uuid) REFERENCES wallets(uuid)
);

-- transactions table
CREATE TABLE transactions (
    txid TEXT PRIMARY KEY,
    block_hash TEXT,
    block_height INTEGER,
    amount REAL,
    fee REAL,
    from_addr TEXT,
    to_addr TEXT,
    confirmations INTEGER DEFAULT 0,
    timestamp DATETIME,
    category TEXT,               -- "send" | "receive" | "stake"
    notes TEXT
);

-- utxos table (cached)
CREATE TABLE utxos (
    txid TEXT,
    vout INTEGER,
    address TEXT,
    amount REAL,
    scriptPubKey TEXT,
    confirmations INTEGER,
    spent BOOLEAN DEFAULT 0,
    PRIMARY KEY (txid, vout)
);
```

### 2. NFXAPI (RPC Client)

**File:** `src/nfxapi.cpp`

Handles all communication with the NFX node:

```cpp
class NFXAPI : public QObject {
    Q_OBJECT

public:
    NFXAPI(const QString& rpcUrl = "http://localhost:18332",
           const QString& user = "",
           const QString& password = "",
           QObject* parent = nullptr);
    ~NFXAPI();

    // Connection
    bool Connect();
    void Disconnect();
    bool IsConnected() const;

    // Blockchain
    QVariantMap GetBlockchainInfo();
    QVariantMap GetBlock(const QString& hash, bool verbose = true);
    int GetBlockCount();

    // Transactions
    QByteArray SendRawTransaction(const QByteArray& rawTx);
    QVariantMap GetTransaction(const QString& txid);
    QList<QVariantMap> GetMempool();

    // Balance
    uint64_t GetBalance(const QString& address);
    QList<QVariantMap> ListUnspent(const QString& address);

    // Wallet RPC (if wallet enabled on node)
    QString CreateWallet(const QString& name);
    void LoadWallet(const QString& name);
    QVariantMap GetWalletInfo();

    // Async
    QFuture<QVariantMap> GetBlockchainInfoAsync();
    QFuture<uint64_t> GetBalanceAsync(const QString& address);

signals:
    void connected();
    void disconnected();
    void error(const QString& message);
    void newBlock(const QVariantMap& block);

private:
    QNetworkAccessManager* network_;
    QUrl rpcUrl_;
    QByteArray authHeader_;
    int requestId_ = 1;

    QJsonObject Call(const QString& method, const QJsonArray& params = {});
    QNetworkReply* CallAsync(
        const QString& method,
        const QJsonArray& params,
        std::function<void(QJsonObject)> callback
    );

    void HandleReply(QNetworkReply* reply);
};
```

**Example RPC call (internal):**

```cpp
QVariantMap NFXAPI::GetBlockchainInfo() {
    QJsonObject request = {
        {"jsonrpc", "1.0"},
        {"id", requestId_++},
        {"method", "getblockchaininfo"},
        {"params", QJsonArray()}
    };

    QJsonObject response = Call("getblockchaininfo");
    QVariantMap result;

    if (response.contains("result")) {
        result = response["result"].toVariant().toMap();
    } else {
        qWarning() << "RPC error:" << response["error"].toString();
    }

    return result;
}
```

### 3. NodeDiscovery

**File:** `src/nodediscovery.cpp`

Automatically finds and connects to local or remote nodes:

```cpp
class NodeDiscovery : public QObject {
    Q_OBJECT

public:
    explicit NodeDiscovery(QObject* parent = nullptr);

    // Discover nodes
    QList<NodeInfo> DiscoverLocal();      // Scan localhost common ports
    QList<NodeInfo> DiscoverDNS();        // Query DNS seeds
    QList<NodeInfo> DiscoverFromPeers();  // Ask connected peers

    // Connection testing
    bool TestConnection(const NodeInfo& node);

    // Bookmark management
    void AddBookmark(const NodeInfo& node);
    QList<NodeInfo> GetBookmarks() const;

signals:
    void nodeDiscovered(const NodeInfo& node);
    void discoveryFinished(const QList<NodeInfo>& nodes);

private:
    QList<NodeInfo> cachedNodes_;
    QList<NodeInfo> bookmarks_;
    QTimer* discoverTimer_;
};
```

### 4. Transaction Table Model

**File:** `src/widgets/transactiontablemodel.cpp`

```cpp
class TransactionTableModel : public QAbstractTableModel {
    Q_OBJECT

public:
    enum Columns {
        Date = 0,
        Description,
        Category,
        Amount,
        Confirmations,
        TXID,
        COUNT
    };

    TransactionTableModel(QObject* parent = nullptr);

    // QAbstractItemModel overrides
    int rowCount(const QModelIndex& parent = QModelIndex()) const override;
    int columnCount(const QModelIndex& parent = QModelIndex()) override;
    QVariant data(const QModelIndex& index, int role = Qt::DisplayRole) const override;
    QVariant headerData(int section, Qt::Orientation orientation, int role = Qt::DisplayRole) const override;

    // Data management
    void addTransaction(const Transaction& tx);
    void clear();
    void refresh();  // Reload from database

private:
    QList<Transaction> transactions_;  // In-memory cache
    WalletDatabase* db_;
};
```

### 5. Logger

**File:** `src/logger.cpp`

Centralized logging with categories:

```cpp
enum class LogCategory {
    General,
    Network,
    RPC,
    Wallet,
    Staking,
    AI,
    GUI
};

class Logger : public QObject {
    Q_OBJECT

public:
    static Logger* Instance();

    void SetLogLevel(LogLevel level);
    void SetCategories(QStringList categories);

    void Log(LogCategory cat, LogLevel level, const QString& message);

    // Convenience macros
    #define LOG_DEBUG(cat, msg) Logger::Instance()->Log(cat, Debug, msg)
    #define LOG_INFO(cat, msg)  Logger::Instance()->Log(cat, Info, msg)
    #define LOG_WARN(cat, msg)  Logger::Instance()->Log(cat, Warn, msg)
    #define LOG_ERROR(cat, msg) Logger::Instance()->Log(cat, Error, msg)

signals:
    void logMessage(const QString& timestamp, const QString& category, const QString& level, const QString& message);

private:
    Logger();
    ~Logger();

    QFile* logFile_;
    QTextStream* stream_;
    LogLevel minLevel_ = Info;
    QStringList enabledCategories_;
};
```

## Configuration

Wallet reads config from `~/.nfx/config.toml` (same as node).

**Wallet-specific settings:**

```toml
[wallet]
enabled = true
walletdir = "~/.nfx/wallets"
default_wallet = ""  # Auto-load this wallet
auto_backup = true   # Backup to cloud (if enabled)
backup_interval = "7d"  # Weekly
show_balance_precision = 8  # Decimal places
fiat_currency = "USD"  # Price display
use_custom_node = false  # Use external node
custom_node_url = ""  # if use_custom_node
```

### Settings Dialog

Implemented in `SettingsPage.qml`:

```qml
Page {
    title: "Settings"

    FormLayout {
        ComboBox {
            model: ["Testnet", "Mainnet"]
            onActivated: config.network = currentText.toLowerCase()
        }

        TextField {
            id: rpcHost
            placeholderText: "RPC Host"
            text: config.rpc_host
        }

        SpinBox {
            id: feeSpin
            from: 0.001; to: 1.0; step: 0.001
            value: config.default_fee
        }

        Switch {
            id: startMinimized
            text: "Start minimized to tray"
            checked: config.start_minimized
        }

        Button {
            text: "Encrypt Wallet"
            onClicked: wallet.encrypt()
        }

        Button {
            text: "Backup Wallet"
            onClicked: wallet.backup()
        }
    }
}
```

## Security

### Private Key Storage

- **Encryption:** AES-256-CBC with PBKDF2 key derivation (100,000 iterations)
- **Storage:** Encrypted in SQLite database
- **Memory:** Keys kept only in RAM, zeroed on exit
- **Backup:** Encrypted JSON export

```cpp
// Encryption example
QByteArray encrypt(const QByteArray& plaintext, const QByteArray& password) {
    QByteArray salt = QCryptographicHash::randomBytes(32);
    QByteArray key = PBKDF2(password, salt, 100000, 32);  // 256-bit key

    QCipherEngine cipher(QCipherEngine::AES256, QCipherEngine::CBC);
    cipher.setKey(key);
    cipher.setIV(QCryptographicHash::hash(salt, QCryptographicHash::Sha256).left(16));

    return salt + cipher.encrypt(plaintext);
}
```

### Address Validation

```cpp
bool NFXAPI::ValidateAddress(const QString& address) const {
    // Check format: NFX1 + base58check
    QRegularExpression re("^NFX1[1-9A-HJ-NP-Za-km-z]+$");
    if (!re.match(address).hasMatch()) return false;

    // Verify checksum
    QString body = address.left(address.length - 8);  // Excluding checksum
    QString checksum = address.right(8);
    QString computed = Checksum(body);

    return checksum == computed;
}
```

### Transaction Signing

```cpp
QByteArray Wallet::SignTransaction(
    const QByteArray& rawTx,
    const QByteArray& privkey
) {
    // Parse raw transaction
    CTransaction tx;
    if (!tx.deserialize(rawTx)) {
        throw std::runtime_error("Invalid transaction");
    }

    // Sign each input
    for (auto& input : tx.vin) {
        // Create signature hash (SIGHASH_ALL)
        uint256 hash = SignatureHash(input, tx);

        // Sign with ECDSA (or SPHINCS+)
        QByteArray sig = crypto::SignHash(hash, privkey);

        // Append SIGHASH flag
        sig.append(0x01);  // SIGHASH_ALL

        input.scriptSig = std::vector<uint8_t>(
            sig.begin(), sig.end()
        );
    }

    // Serialize signed transaction
    return tx.serialize();
}
```

## Staking (PoS/PoES)

### Staking Dashboard

The staking page enables users to delegate their NFX tokens:

```cpp
class StakingManager : public QObject {
    Q_OBJECT

public:
    struct StakeInfo {
        QString address;
        uint64_t amount;
        uint64_t rewards;
        QDateTime start_time;
        QDateTime unlock_time;
        bool active;
    };

    StakingManager(NFXAPI* rpc, WalletManager* wallet);

    // Stake operations
    bool Stake(const QString& address, uint64_t amount);
    bool Unstake(const QString& address, uint64_t amount);
    QList<StakeInfo> GetActiveStakes() const;
    uint64_t CalculateRewards(const QString& address, QDateTime since) const;

    // Claim rewards
    bool ClaimRewards(const QString& address);

signals:
    void stakingStatusChanged();
    void rewardsUpdated(uint64_t total);
};
```

**Staking Process:**

1. User selects address and amount in UI
2. Wallet creates `staketoken` transaction (special opcode)
3. Transaction broadcast to network
4. PoES validator includes in block
5. Rewards accrue per block (adjusted by stake weight)
6. After lock period (default 7 days), can withdraw

### UI Example (QML)

```qml
Page {
    property var stakeInfo: ({})  // Populated from C++

    Column {
        spacing: 20
        padding: 30

        Card {
            title: "Staking Overview"
            Label { text: "Staked: " + formatAmount(stakeInfo.staked) }
            Label { text: "Available: " + formatAmount(stakeInfo.available) }
            ProgressBar {
                from: 0; to: 1
                value: stakeInfo.staked / (stakeInfo.staked + stakeInfo.available)
            }
        }

        Card {
            title: "Stake Funds"
            ComboBox { model: wallet.addresses }
            SpinBox { from: 1000; to: 9999999999 }  // Min 1000 NFX

            Button {
                text: "Stake"
                enabled: spinbox.value >= 1000 && spinbox.value <= available
                onClicked: wallet.stake(combo.currentText, spinbox.value)
            }
        }

        Card {
            title: "Active Stakes"
            ListView {
                model: stakeInfo.active_stakes
                delegate: Row {
                    Text { text: model.address + " - " + formatAmount(model.amount) }
                    Text { text: "Rewards: " + formatAmount(model.rewards) }
                }
            }
        }
    }
}
```

## Hypernode Monitoring

**File:** `src/guardianspage.qml`

Monitor guardian/hypernode performance:

```qml
Page {
    title: "Guardian Nodes"

    ListView {
        model: guardianModel

        delegate: Card {
            RowLayout {
                Image { source: "icons/guardian.png" }
                Column {
                    Text { text: model.name; font.bold: true }
                    Text { text: "Status: " + model.status }
                    ProgressBar {
                        from: 0; to: 100
                        value: model.uptime_percent
                    }
                }
                Button {
                    text: "Manage"
                    onClicked: showGuardianDetails(model.id)
                }
            }
        }
    }

    Button {
        text: "Register New Guardian"
        onClicked: showRegistrationDialog()
    }
}
```

### Guardian Registration (C++)

```cpp
bool Wallet::RegisterGuardian(
    const QString& address,
    uint64_t stake_amount
) {
    // Must have minimum stake (configurable, default 10,000 NFX)
    if (stake_amount < MIN_GUARDIAN_STAKE) {
        emit error("Insufficient stake. Minimum: 10,000 NFX");
        return false;
    }

    // Create special transaction with OP_REGISTER_GUARDIAN
    CTransaction tx;
    tx.vin = ...;      // Inputs from address
    tx.vout = ...;
    tx.nVersion = 2;
    tx.nType = TX_GUARDIAN_REGISTRATION;

    // Add registration data
    GuardianRegistration reg;
    reg.address = address.toStdString();
    reg.stake_amount = stake_amount;
    reg.timestamp = GetTime();
    reg.signature = SignWithPrivKey(reg);  // Sign

    tx.vExtra.push_back(serialize(reg));

    return Broadcast(tx);
}
```

## Address Management

### Address Generator (BIP-44)

```cpp
class AddressBookModel : public QAbstractTableModel {
    Q_OBJECT

public:
    enum Columns { Address = 0, Label, Balance, COUNT };

    QString CreateAddress(const QString& label = "My Address");
    void SetLabel(const QString& address, const QString& label);
    void DeleteAddress(const QString& address);

    // Reuse existing (import)
    QString ImportAddress(const QString& wif, const QString& label);

    // Watch-only (no private key)
    QString ImportWatchOnly(const QString& address, const QString& label);

private:
    struct AddressEntry {
        QString address;
        QString label;
        QString pubkey;     // Hex
        QString wif;        // Encrypted storage
        bool is_watchonly;
        bool is_change;     // Change address
    };

    QList<AddressEntry> addresses_;
    WalletDatabase* db_;
};
```

### Address Validation

```qml
TextField {
    id: addressField
    placeholderText: "NFX1..."

    validator: RegularExpressionValidator {
        regularExpression: /^NFX1[1-9A-HJ-NP-Za-km-z]+$/
    }

    onTextChanged: {
        if (acceptableInput) {
            validationText.text = "âœ“ Valid address"
            validationText.color = "green"
        } else {
            validationText.text = "âœ— Invalid checksum or format"
            validationText.color = "red"
        }
    }
}
```

## Transaction History

### Transaction Model

```cpp
class TransactionModel : public QSortFilterProxyModel {
    Q_OBJECT

public:
    enum Role {
        TxIDRole = Qt::UserRole + 1,
        AmountRole,
        ConfirmationsRole,
        CategoryRole,  // "send" | "receive" | "stake"
        TimeRole,
        AddressRole
    };

    TransactionModel(QObject* parent = nullptr);

    // Filtering
    void SetFilterAddress(const QString& address);
    void SetFilterCategory(const QString& category);  // "send" or "receive"
    void SetShowZeroConf(bool show);

    // Sorting
    void SortByDate(bool ascending = false);
    void SortByAmount(bool descending = true);
};
```

### QML View

```qml
ListView {
    model: transactionModel

    delegate: RowLayout {
        anchors.fill: parent
        spacing: 10

        // Icon (sent/received)
        Image {
            source: model.category === "send" ?
                    "icons/arrow_up.png" : "icons/arrow_down.png"
        }

        Column {
            Text {
                text: model.address
                font.bold: true
                elide: Text.ElideMiddle
            }
            Text {
                text: Qt.formatDateTime(model.time, "yyyy-MM-dd hh:mm")
                color: "#666"
                font.pixelSize: 12
            }
        }

        Item { Layout.fillWidth: true }  // Spacer

        Column {
            horizontalAlignment: Qt.AlignRight
            Text {
                text: formatAmount(model.amount)
                font.bold: true
                color: model.category === "send" ? "red" : "green"
            }
            Text {
                text: model.confirmations + " confirmations"
                color: model.confirmations < 6 ? "orange" : "green"
                font.pixelSize: 11
            }
        }
    }

    section {
        property: "time_section"  // Group by date
        delegate: Label { text: section }
    }
}
```

## Settings & Preferences

### Application Settings (QSettings)

```cpp
class WalletSettings : public QObject {
    Q_OBJECT

public:
    static WalletSettings* Instance();

    // General
    Q_PROPERTY(QString language MEMBER language_ WRITE setLanguage NOTIFY languageChanged)
    Q_PROPERTY(QString theme MEMBER theme_ WRITE setTheme NOTIFY themeChanged)

    // Display
    Q_PROPERTY(int decimalPlaces MEMBER decimal_places_ WRITE setDecimalPlaces)
    Q_PROPERTY(QString fiatCurrency MEMBER fiat_currency_ WRITE setFiatCurrency)

    // Network
    Q_PROPERTY(QString rpcUrl MEMBER rpc_url_ WRITE setRpcUrl)
    Q_PROPERTY(QString nodeAddress MEMBER node_address_ WRITE setNodeAddress)

    // UI
    Q_PROPERTY(bool minimizeToTray MEMBER minimize_to_tray_ WRITE setMinimizeToTray)
    Q_PROPERTY(bool startMinimized MEMBER start_minimized_ WRITE setStartMinimized)

    void Load();
    void Save();

signals:
    void languageChanged();
    void themeChanged();

private:
    QSettings settings_;
    QString language_ = "en";
    QString theme_ = "dark";
    int decimal_places_ = 8;
    QString fiat_currency_ = "USD";
    // ...
};
```

### Themes (Dark/Light)

```qml
// theme.qml
import QtQuick 2.15

Item {
    property color background: "#1a1f2e"
    property color surface: "#2d3748"
    property color primary: "#3498db"
    property color text: "#e2e8f0"
    property color textSecondary: "#94a3b8"
    property color border: "#4a5568"

    property color success: "#27ae60"
    property color warning: "#f39c12"
    property color danger: "#e74c3c"
}
```

## Internationalization (i18n)

Translation files in `translations/`:

```bash
# Update translations (lupdate)
lupdate nfx-wallet.pro -ts translations/nfx-qt_pt_BR.ts

# Compile translations (lrelease)
lrelease translations/*.ts -qm translations/*.qm

# Load in QML
import QtQuick 2.15
import QtQuick.LocalStorage 2.15

Text {
    text: qsTr("Send")  // Translated
}
```

Switch language dynamically:

```cpp
void Wallet::SetLanguage(const QString& lang) {
    QTranslator* translator = new QTranslator(this);
    QString qmPath = QString(":/translations/nfx-qt_%1.qm").arg(lang);
    if (translator->load(qmPath)) {
        QApplication::installTranslator(translator);
        settings.setValue("language", lang);
    }
}
```

## Coin Control (Advanced)

Allow users to select specific UTXOs for spending:

```cpp
struct CoinControlEntry {
    COutPoint outpoint;     // TXID + vout
    QString address;
    uint64_t amount;
    bool selected = false;
};

class CoinControlDialog : public QDialog {
    Q_OBJECT

public:
    QList<CoinControlEntry> GetSelectedUTXOs() const;

private slots:
    void onSelectAll();
    void onSelectNone();
    void onAmountExact();  // Select UTXOs matching exact amount

signals:
    void selectionChanged();
};
```

Usage:
```cpp
QList<CoinControlEntry> selected = dialog.GetSelectedUTXOs();
txbuilder.SetInputs(selected);
```

## Debug Console

Built-in debugging console (similar to Bitcoin Core's):

```qml
Dialog {
    title: "Debug Console"

    TextArea {
        id: consoleOutput
        readOnly: true
        text: Logger.recentMessages.join("\n")
    }

    TextField {
        id: commandInput
        placeholderText: "Enter command..."

        onAccepted: {
            var result = rpc.call("eval", commandInput.text)
            consoleOutput.append("> " + commandInput.text)
            consoleOutput.append(result)
        }
    }

    ComboBox {
        model: ["getinfo", "getblockcount", "listunspent", "getrawmempool"]
        onActivated: commandInput.text = currentText
    }
}
```

**Supported commands:**
- `getinfo` - Node status
- `getbalance <address>` - Balance
- `gettransaction <txid>` - Transaction details
- `sendrawtransaction <hex>` - Broadcast raw tx
- `decoderawtransaction <hex>` - Decode hex

## Troubleshooting UI Issues

### Blank Screen (QML)

**Cause:** Missing QML import path.

**Fix:**
```bash
# Set QML2 import path
export QML2_IMPORT_PATH=/usr/lib/x86_64-linux-gnu/qml

# Or run with debug output
nfx-wallet --qmljsdebugger=port:1234
```

### Icons Not Showing

```bash
# Recompile Qt resources
cd nfx-wallet
rcc resources.qrc -o qrc_resources.cpp
# Then rebuild project
```

### High CPU Usage

**Cause:** UI refresh too fast (balance polling every 100ms).

**Fix:** Increase refresh interval:

```cpp
// In walletmanager.cpp
refreshTimer_->setInterval(5000);  // 5 seconds
```

## Contributing UI Changes

1. QML changes: Edit `.qml` files, test with `qmlscene`
2. Widget changes: Edit `.ui` files in Qt Designer
3. C++ logic: Follow C++ style guide
4. Translations: Update `translations/*.ts`, run `lupdate`, commit `.ts` files

---

*Next: [Configuration Reference](../guides/configuration.md) | [FAQ](../faq.md) | [API Reference](../api/)*
