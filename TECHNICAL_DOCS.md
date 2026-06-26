# Vyniq Chain — Technical Documentation

> **Audience:** Developers, node operators, and technical integrators.
> This document contains the full technical reference for the Vyniq Chain protocol.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Core Blockchain](#2-core-blockchain)
3. [JSON-RPC API Reference](#3-json-rpc-api-reference)
4. [REST API Reference](#4-rest-api-reference)
5. [WebSocket API](#5-websocket-api)
6. [Wallet System](#6-wallet-system)
7. [P2P Network Layer](#7-p2p-network-layer)
8. [L2 Sequencer Engine](#8-l2-sequencer-engine)
9. [Database Schemas & Persistence](#9-database-schemas--persistence)
10. [Transaction Lifecycle](#10-transaction-lifecycle)
11. [Consensus Mechanism](#11-consensus-mechanism)
12. [Testnet Deployment Guide](#12-testnet-deployment-guide)
13. [Data Structures](#13-data-structures)

---

## 1. Architecture Overview

### 1.1 Intended System Architecture

```
BNB Chain (L1 — Planned Settlement Anchor)
       |
       | Bridge (not yet built)
       |
Vyniq Chain (L2 — Execution, Consensus)
       |
       +-- API Server (TypeScript): REST API, JSON-RPC, WebSocket
       +-- L2 Engine (TypeScript): Sequencer, batch proofs, state management
       +-- Validator Node (Rust): PoW / PoA consensus, P2P gossip
       +-- P2P Network (Rust/libp2p): gossipsub mesh, peer discovery
       +-- Mobile Light Node (Rust): Future lightweight validation
       +-- Mobile Wallet (React Native): Non-custodial wallet app
```

### 1.2 Component Matrix

| Layer | Component | Language | Role |
|-------|-----------|----------|------|
| L1 | BNB Chain | — | Security & bridge settlement (planned) |
| L2 | API Server | TypeScript | Canonical source, REST API, JSON-RPC, WebSocket |
| L2 | Validator Node | Rust | PoW mining, block validation, P2P integration |
| L2 | P2P Network | Rust | Peer discovery, block gossip (libp2p) |
| L2 | L2 Engine | TypeScript | Sequencer, batch proofs, state management |
| L2 | Mobile Light Node | Rust | Mobile-friendly read-only client (future) |

### 1.3 Core Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Signature Scheme | Ed25519 (RFC 8032) | Faster verification than ECDSA; smaller signatures; mobile-efficient |
| Consensus (Phase 1) | PoW (difficulty-targeted) | Simple bootstrap mechanism |
| Consensus (Planned) | Proof-of-Authority | Target for future transition |
| P2P Stack | libp2p (gossipsub, Noise, Yamux) | Battle-tested, modular, mobile-compatible |
| L1 Anchor (Planned) | BNB Chain | Fast finality, low cost, large ecosystem |
| Block Time | 2 seconds (target) | Prototype achieves ~2s on single node |
| Fee Model | 50% burn, 50% to validators | Designed for deflationary pressure |
| Storage (Current) | JSON files | Chosen for development speed and auditability |

---

## 2. Core Blockchain

The core blockchain engine is implemented in Rust (`core-chain/src/lib.rs`, ~300 lines). It handles:

- **Transaction creation and validation** — Ed25519 signature verification, nonce checking, balance checks
- **Block production** — PoW mining with configurable difficulty target
- **Chain validation** — Merkle root verification, parent hash linking, block index ordering
- **Wallet operations** — Ed25519 key generation, address derivation

### 2.1 Block Structure

```typescript
{
  index: number;
  timestamp: number;
  transactions: Transaction[];
  previousHash: string;
  hash: string;
  nonce: number;
}
```

### 2.2 Transaction Structure

```typescript
{
  sender: string;
  receiver: string;
  amount: number;
  nonce: number;
  hash: string;
  signature: string;
  publicKey: string;
  fee: number;
}
```

### 2.3 VynAccount Structure

```typescript
{
  address: string;
  balance: number;
  staked: number;
  pendingRewards: number;
  nonce: number;
  txHistory: string[];
}
```

### 2.4 Configuration

```json
{
  "networkName": "Vyniq Testnet",
  "chainId": 4201,
  "rpcPort": 3000,
  "nodeName": "vyniq-testnet-1",
  "explorerUrl": "https://explorer.vyniq.xyz",
  "faucetUrl": "https://faucet.vyniq.xyz",
  "symbol": "VYN",
  "decimals": 0
}
```

---

## 3. JSON-RPC API Reference

### 3.1 Endpoint

```
Local:    http://localhost:3000/rpc
Testnet:  https://rpc.vyniq.xyz/rpc
```

**Headers:** `Content-Type: application/json`

All requests use HTTP POST with a JSON-RPC 2.0 body.

### 3.2 Supported Methods

#### eth_chainId

Returns the chain ID.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_chainId",
  "params": []
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x1069"
}
```

`0x1069` = 4201 (Vyniq Testnet).

#### net_version

Returns the network version.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "net_version",
  "params": []
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "4201"
}
```

#### eth_blockNumber

Returns the latest block height.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_blockNumber",
  "params": []
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x8"
}
```

#### eth_getBlockByNumber

Returns block details by block number.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| blockTag | `string` | Hex block number, `"latest"`, or `"earliest"` |
| fullTxs | `boolean` | If true, returns full transaction objects |

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_getBlockByNumber",
  "params": ["0x0", true]
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "number": "0x0",
    "hash": "0b911e48e07367149a45cd745d6e04a2b7eef2b5e3542872ee68d536c06e914b",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp": "0x1781086500",
    "transactions": [],
    "transactionCount": 0,
    "nonce": "0x0",
    "gasLimit": "0x0",
    "gasUsed": "0x0",
    "size": "0x0"
  }
}
```

#### eth_getTransactionByHash

Returns transaction details by transaction hash.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| hash | `string` | Transaction hash |

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "hash": "abc123...",
    "blockHash": "0b911e48...",
    "blockNumber": "0x5",
    "from": "a1b2c3d4e5f6...",
    "to": "f6e5d4c3b2a1...",
    "value": "0xa",
    "gas": "0x0",
    "gasPrice": "0x0",
    "nonce": "0x0",
    "input": "0x",
    "v": "0x0",
    "r": "0x",
    "s": "0x"
  }
}
```

#### eth_getBalance

Returns the balance of an address (in VYN units, no decimals).

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| address | `string` | Address (40 hex chars) |
| blockTag | `string` | (Optional) Block number or tag |

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_getBalance",
  "params": ["a1b2c3d4e5f6..."]
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0xbe"
}
```

#### eth_getTransactionCount

Returns the nonce for an address.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| address | `string` | Address |

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x3"
}
```

#### eth_getBlockTransactionCountByNumber

Returns the transaction count in a block.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| blockTag | `string` | Hex block number, `"latest"`, or `"earliest"` |

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x2"
}
```

#### eth_sendRawTransaction

Submits a signed transaction to the network.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| tx | `object` | Transaction object |

Transaction object fields: `sender`, `receiver`, `amount`, `nonce`, `signature`, `publicKey`, `fee`.

**Response:** Transaction hash (hex string).

#### eth_gasPrice

Returns the current gas price.

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x0"
}
```

#### eth_estimateGas

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x0"
}
```

#### web3_clientVersion

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "VyniqChain/4201/v1.0.0"
}
```

#### net_peerCount

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x0"
}
```

### 3.3 Health Check

```
GET /rpc/health
```

```json
{
  "status": "ok",
  "chainHeight": 8,
  "peerCount": 0,
  "latestBlock": "2602..."
}
```

### 3.4 Network Info

```
GET /rpc/info
```

```json
{
  "networkName": "Vyniq Testnet",
  "chainId": 4201,
  "rpcPort": 3000,
  "nodeName": "vyniq-testnet-1",
  "latestBlock": 8,
  "genesisHash": "0b91...",
  "totalSupply": 10000000000
}
```

---

## 4. REST API Reference

### 4.1 Chain Endpoints

#### GET /chain/info

```json
{
  "chain": "Vyniq Chain",
  "blocks": 42,
  "transactions": 156,
  "mempool": 3,
  "genesisHash": "0b911e48e07367149a45cd745d6e04a2b7eef2b5e3542872ee68d536c06e914b",
  "latestBlock": { "index": 41, "hash": "0000abcd...", "timestamp": 1781242280302 },
  "accounts": 15,
  "wallets": 10,
  "vyn": { "maxSupply": 10000000000, "totalMinted": 1470, "totalBurned": 0 }
}
```

#### GET /status

Returns detailed chain status with VYN supply info.

#### GET /blocks

Returns all blocks with summary info.

#### GET /block/:index

Returns a full block by numeric index.

#### GET /block/hash/:hash

Returns a full block by hash.

### 4.2 Transaction Endpoints

#### GET /transactions

Returns all confirmed and pending transactions.

#### GET /transaction/:hash

Look up a single transaction by hash.

#### POST /transaction/signed

Submit a signed transaction.

### 4.3 Wallet Endpoints

#### POST /wallet/create

Generate a new wallet (returns address and public key; private key returned only at creation time).

#### GET /wallet/:address

Get wallet info with chain-computed balance and VYN account state.

#### GET /wallet/:address/history

Get transaction history for an address.

#### GET /wallets

List all created wallets (public keys only).

### 4.4 Network Endpoints

#### GET /peers

List connected P2P peers.

#### GET /network/status

P2P network status (peer count, topics, listen addresses).

### 4.5 Faucet Endpoint

#### GET /faucet/claim?address=<address>

Claims 10 VYN testnet tokens (24-hour cooldown per address). Rate limited.

---

## 5. WebSocket API

### 5.1 Endpoint

```
ws://localhost:3000/ws
wss://rpc.vyniq.xyz/ws
```

### 5.2 Supported Subscriptions

| Method | Parameters | Description |
|--------|------------|-------------|
| `eth_subscribe` | `"newHeads"` | Receive block headers as they are mined |
| `eth_subscribe` | `"newPendingTransactions"` | Receive transaction hashes entering mempool |
| `eth_unsubscribe` | subscription ID | Unsubscribe from a notification |

### 5.3 Subscription Example

**Client → Server:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_subscribe",
  "params": ["newHeads"]
}
```

**Server → Client (on new block):**
```json
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x1",
    "result": {
      "number": "0xa",
      "hash": "0000abcd...",
      "parentHash": "0000ef01...",
      "timestamp": "0x1781086600",
      "transactions": ["txhash1", "txhash2"]
    }
  }
}
```

---

## 6. Wallet System

### 6.1 Key Generation

Wallets use Ed25519 key pairs:
- **Private key**: PKCS#8 DER-encoded, stored as hex
- **Public key**: SPKI DER-encoded, stored as hex
- **Address**: SHA-256 of hex public key, first 40 hex characters

### 6.2 Transaction Signing

`signData(data, privateKeyHex)` signs a transaction payload using Ed25519:
- Message: `sender + receiver + amount + nonce + fee` (concatenated strings)
- Returns hex-encoded signature

`verifySignature(tx)` verifies an Ed25519 signature on a transaction.

### 6.3 Signed Transaction Flow

1. Wallet generates key pair via `/wallet/create`
2. Client signs `sender+receiver+amount+nonce+fee` with private key
3. Client submits to `POST /transaction/signed` with `{sender, receiver, amount, nonce, signature, publicKey, fee}`
4. Server validates: signature, nonce, balance, duplicate hash
5. Transaction enters mempool
6. On mine: balances are applied, nonce is incremented

### 6.4 Signature Verification Performance

| Layer | Implementation | Performance |
|-------|---------------|-------------|
| Core Blockchain (Rust) | `ed25519-dalek` crate | ~60 µs per signature |
| API Server (TypeScript) | Node.js `crypto` module | ~80 µs per signature |
| Mobile Wallet (React Native) | `tweetnacl` package | ~120 µs per signature |
| P2P Network (Rust) | libp2p Ed25519 identity | Built into protocol |

---

## 7. P2P Network Layer

### 7.1 Components

#### P2P Node (`p2p-network/src/lib.rs`)

Core libp2p wrapper managing connections, gossipsub, and custom protocols:
- `createP2PNode()` — Returns a new P2P node instance
- `node.start()` — Initializes libp2p, subscribes to topics, sets up handlers
- `node.stop()` — Gracefully shuts down connections

#### Message Types

| Type | Topic | Description |
|------|-------|-------------|
| `Transaction` | `/vyniq/transactions/1.0.0` | Signed transaction broadcast |
| `Block` | `/vyniq/blocks/1.0.0` | Mined block broadcast |

#### Chain Sync

Custom protocol `/vyniq/sync/1.0.0` for requesting missing blocks:
- `syncRequest(fromBlock, toBlock)` — Request a range of blocks from a peer
- `syncResponse(blocks)` — Respond with the requested block range
- `validateIncomingBlock(block, latestBlock)` — Validate a received block

### 7.2 Node Configuration

```json
{
  "nodeName": "vyniq-node-1",
  "p2pPort": 9837,
  "apiPort": 3000,
  "bootstrapPeers": [],
  "enableMdns": true,
  "maxPeers": 50,
  "reconnectInterval": 5000
}
```

### 7.3 Events

| Event | Trigger | Handler Action |
|-------|---------|----------------|
| `transaction` | Gossipsub message received | Add to local mempool if new |
| `block` | Gossipsub block received | Validate and append to chain |
| `peerConnected` | New peer connection | Track in peer list |
| `peerDisconnected` | Peer disconnects | Remove from peer list |
| `syncComplete` | Chain sync finished | Log sync results |

### 7.4 Message Flow

**Transaction Propagation:**
1. User submits tx via `POST /transaction/signed`
2. API validates, adds to mempool
3. P2P broadcasts to `vyniq-transactions` topic
4. All connected peers receive and add to their mempools

**Block Propagation:**
1. Miner calls `POST /mine`
2. Block is created and added to local chain
3. P2P broadcasts to `vyniq-blocks` topic
4. Peers validate and append to their chains

**Chain Synchronization:**
1. New peer connects via bootstrap
2. Peer's chain height is compared
3. If remote is ahead, `/vyniq/sync/1.0.0` protocol requests missing blocks
4. Blocks are validated and applied sequentially

### 7.5 Dependencies

- `libp2p` — Core P2P networking
- `@libp2p/tcp` — TCP transport
- `@chainsafe/libp2p-noise` — Noise encryption
- `@chainsafe/libp2p-yamux` — Stream multiplexing
- `@chainsafe/libp2p-gossipsub` — Pub/sub messaging
- `@libp2p/bootstrap` — Bootstrap peer discovery
- `@libp2p/identify` — Peer identification
- `@multiformats/multiaddr` — Multiaddress handling

---

## 8. L2 Sequencer Engine

The L2 sequencer (`l2-engine/`, ~600+ lines TypeScript) handles:
- Batch production at 3-second intervals
- State management and state root computation
- Bridge event tracking (proof-of-concept, not production-ready)
- Integration with the API server for block production

Key characteristics:
- **Batch interval:** 3 seconds
- **State persistence:** JSON files
- **Validation pipeline:** Transaction signature verification, nonce checks, balance checks, deduplication

---

## 9. Database Schemas & Persistence

### 9.1 Storage Files

All data is stored as JSON files in the `data/` directory:

| File | Content | Purpose |
|------|---------|---------|
| `data/chain.json` | All blocks | Block storage, chain state |
| `data/accounts.json` | Wallet balances, nonces, history | Account state |
| `data/wallets.json` | Public keys | Wallet index |
| `data/mempool.json` | Pending transactions | Transaction pool |
| `data/vyn-state.json` | Token mint/burn state | VYN token state |
| `data/server.lock` | Lock file | Prevents duplicate server instances |

### 9.2 Data Persistence Notes

- All data reloads automatically on server restart
- Storage is currently JSON-based; SQLite migration is planned
- No external database dependency required for current operation

---

## 10. Transaction Lifecycle

### 10.1 Flow

1. User generates Ed25519 keypair (client-side)
2. User creates transaction `{sender, receiver, amount, fee, nonce}`
3. User signs with private key
4. Signed transaction submitted via REST API or WebSocket
5. API Server validates: address format, amount > 0, sufficient balance, nonce uniqueness, signature validity, deduplication, self-send check
6. Validated transaction enters mempool
7. Block is mined (PoW) or proposed (sequencer mode)
8. Block appended to chain with Merkle root and parent hash
9. Block gossiped through P2P network (if other peers are connected)

### 10.2 Validation Rules

| Check | Description |
|-------|-------------|
| Address format | Valid 40-hex character address |
| Amount > 0 | Transaction value must be positive |
| Sufficient balance | Sender must have adequate VYN balance |
| Nonce uniqueness | Nonce must match sender's current nonce |
| Signature validity | Ed25519 signature must verify against public key |
| Deduplication | Transaction hash must not already exist |
| Self-send | Sender cannot equal receiver |

---

## 11. Consensus Mechanism

### 11.1 Phase 1: Proof of Work (Current)

- Difficulty-targeted PoW mining
- Configurable difficulty
- ~2 second block time on single node
- SHA-256 based mining
- Single-node setup (current); multi-validator not yet deployed

### 11.2 Phase 2: Proof of Authority (Planned)

- 7–21 authorized validators
- BFT security model (f < n/2)
- Block rotation among validators
- Slashing conditions for misbehavior

### 11.3 Phase 3: Hybrid + Mobile (Research)

- Gradual inclusion of mobile light nodes
- Delegated validation for resource-constrained devices
- Economic finality model

### 11.4 Token Emission Schedule (Current Testnet)

| Property | Value |
|----------|-------|
| Max Supply | 10,000,000,000 VYN |
| Initial Block Reward | 50 VYN per block |
| Halving Interval | Every 210,000 blocks (~4.9 years at 2s block time) |
| Fee Burn Rate | 50% of transaction fees |
| Current Testnet Supply | ~1,470 VYN minted (single node) |

---

## 12. Testnet Deployment Guide

### 12.1 Prerequisites

- Ubuntu 22.04 LTS or later (VPS)
- Node.js 18+
- PM2 process manager
- Nginx (for reverse proxy)
- SSL certificate (Let's Encrypt)
- Domains: `rpc.vyniq.xyz`, `explorer.vyniq.xyz`, `faucet.vyniq.xyz`

### 12.2 Quick Start

```bash
# Install dependencies
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs git nginx
npm install -g pm2

# Clone and build
git clone https://github.com/vyniq/vyniq-chain.git
cd vyniq-chain/api-server
npm install
npx tsc

# Start with PM2
pm2 start dist/index.js --name "vyniq-api" -- -p 3000
pm2 save
pm2 startup
```

### 12.3 DNS Configuration

| Domain | Type | Value |
|--------|------|-------|
| `rpc.vyniq.xyz` | A | `<VPS_IP>` |
| `explorer.vyniq.xyz` | A | `<VPS_IP>` |
| `faucet.vyniq.xyz` | A | `<VPS_IP>` |

### 12.4 Security

```bash
sudo ufw allow 22/tcp && sudo ufw allow 80/tcp && sudo ufw allow 443/tcp && sudo ufw enable
sudo apt install -y fail2ban
```

### 12.5 Multi-Node Network

**Node 1 (bootstrap):**
```json
{ "nodeName": "validator-1", "p2pPort": 9837, "apiPort": 3000 }
```

**Node 2 (connects to Node 1):**
```json
{ "nodeName": "validator-2", "p2pPort": 9838, "apiPort": 3001,
  "bootstrapPeers": ["/ip4/127.0.0.1/tcp/9837/p2p/PeerId1"] }
```

### 12.6 Verification Checklist

- Server starts without errors
- RPC responds on port 3000
- SSL certificate valid
- `eth_chainId` returns `0x1069`
- `eth_blockNumber` returns current height
- `eth_getBalance` returns correct balance
- Transaction submission via `eth_sendRawTransaction`
- Block queries via `eth_getBlockByNumber`
- Transaction queries via `eth_getTransactionByHash`
- Explorer loads and shows data
- Wallet app connects via RPC
- `GET /rpc/health` returns 200

---

## 13. Data Structures

### 13.1 Block

```typescript
interface Block {
  index: number;
  timestamp: number;
  transactions: Transaction[];
  previousHash: string;
  hash: string;
  nonce: number;
}
```

### 13.2 Transaction

```typescript
interface Transaction {
  sender: string;
  receiver: string;
  amount: number;
  nonce: number;
  hash: string;
  signature: string;
  publicKey: string;
  fee: number;
}
```

### 13.3 VynAccount

```typescript
interface VynAccount {
  address: string;
  balance: number;
  staked: number;
  pendingRewards: number;
  nonce: number;
  txHistory: string[];
}
```

---

## Appendix A: Repository Map

| Component | Path | Lines | Language |
|-----------|------|-------|----------|
| Blockchain Core | `core-chain/src/lib.rs` | 297 | Rust |
| P2P Network | `p2p-network/src/lib.rs` | 128 | Rust |
| Validator Node | `validator-node/src/main.rs` | 140 | Rust |
| Mobile Light Node | `mobile-light-node/src/main.rs` | 5 | Rust |
| API Server | `api-server/src/index.ts` | ~1,700 | TypeScript |
| L2 Engine | `l2-engine/` (multiple files) | ~600 | TypeScript |
| Faucet | `faucet/src/index.ts` | 330 | TypeScript |
| Block Explorer | `apps/vyniq-explorer/` | ~1,000 | TypeScript/React |
| Mobile Wallet | `apps/vyniq-wallet/` | ~2,000 | TypeScript/React Native |
| E2E Tests | `e2e-test.js`, `e2e-full.js`, `send-tx.js` | ~130 | JavaScript |
| Integration Tests | `l2-engine/test/integration.ts` | 169 | TypeScript |
| RPC Tests | `rpc_test.json` | — | JSON |

## Appendix B: Current Testnet Parameters

| Parameter | Value |
|-----------|-------|
| RPC Endpoint | `https://rpc.vyniq.xyz/rpc` |
| Chain ID | 4201 |
| Block Time | ~2 seconds |
| Transaction Throughput (single node) | ~500 tps (untested under load) |
| Validators | 1 (single-node) |
| Faucet | 10 VYN/claim, 24h cooldown |
| Wallet Compatibility | MetaMask (via JSON-RPC) |
