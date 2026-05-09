# Glossary

## Consensus

**PoES (Proof of Exponential Security)**
> A novel consensus mechanism where the security of the network increases exponentially with the number of honest nodes. Mathematical formula: `S(t) = 1 - e^(-kÂ·hÂ·nÂ·t)`.

**PoW (Proof of Work)**
> Miners compete to solve cryptographic puzzles; first to solve adds block and receives reward.

**PoS (Proof of Stake)**
> Validators chosen based on amount of tokens staked; higher stake = higher probability of block creation.

## Cryptography

**SHA-3 (Keccak)**
> Secure Hash Algorithm 3, quantum-resistant hash function used by NFX Chain.

**SPHINCS+**
> Stateless hash-based signature scheme resistant to quantum computer attacks.

**BIP-39**
> Bitcoin Improvement Proposal 39: Mnemonic sentence for deterministic key generation.

**BIP-44**
> Multi-account hierarchy for deterministic wallets.

## Network

**Hypernode**
> A specially staked full node (â‰¥10,000 NFX) that participates in AI governance and earns rewards.

**Guardian**
> Synonym for hypernode in guardian/validator role.

**P2P (Peer-to-Peer)**
> Decentralized network where nodes communicate directly without central server.

**DNS Seed**
> DNS record returning a list of node IPs for initial bootstrapping.

## Transactions & Blocks

**UTXO (Unspent Transaction Output)**
> Model where each transaction consumes and creates discrete outputs.

**Mempool**
> Pool of unconfirmed transactions waiting to be included in a block.

**Coinbase Transaction**
> First transaction in a block, creating new coins (block reward).

**Confirmation**
> Number of blocks added after a transaction; 6 confirmations = final.

## Governance

**AI Governance**
> Neural network (tiny-dnn) that scores blocks for validity, detecting anomalies or attacks.

**Threshold Valid**
> AI confidence threshold above which block is automatically accepted (default: 0.80).

**Staking**
> Locking tokens to participate in PoS/PoES consensus and earn rewards.

**Maturity**
> Number of blocks before staked tokens become spendable again.

## Development

**SDK (Software Development Kit)**
> Libraries and tools for building applications that interact with NFX Chain.

**RPC (Remote Procedure Call)**
> JSON over HTTP API for node control and queries.

**WebSocket**
> Full-duplex communication channel for real-time notifications.

**Gas**
> Computational cost unit for smart contract execution.

**TOML**
> Tom's Obvious, Minimal Language â€” configuration file format.

## Economics

**Inflation**
> New token issuance via block rewards; NFX has no inflation after block 1,000,000 (PoES secures without rewards).

**Fee Market**
> Transaction fees paid to miners/validators; higher fee = faster inclusion.

**Total Supply**
> Fixed 100,000,000 NFX (10^16 smallest units).

---

*Missing term? Open an issue or PR.*
