# Vyniq Chain — Technical Documentation v2.0

> **Audience:** Developers, node operators, SocialFi integrators, and technical evaluators.
> This document contains the full technical reference for the Vyniq Chain protocol v2.0,
> including the integrated SocialFi ecosystem.

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
9. [SocialFi Architecture](#9-socialfi-architecture)
10. [User Data Flow](#10-user-data-flow)
11. [Wallet-to-Social Linking](#11-wallet-to-social-linking)
12. [Token Reward Flow](#12-token-reward-flow)
13. [Reputation Engine](#13-reputation-engine)
14. [Social APIs](#14-social-apis)
15. [Mobile Synchronization](#15-mobile-synchronization)
16. [Future AI Moderation](#16-future-ai-moderation)
17. [Database Schemas & Persistence](#17-database-schemas--persistence)
18. [Transaction Lifecycle](#18-transaction-lifecycle)
19. [Consensus Mechanism](#19-consensus-mechanism)
20. [Testnet Deployment Guide](#20-testnet-deployment-guide)
21. [Data Structures](#21-data-structures)

---

## 1. Architecture Overview

### 1.1 Intended System Architecture (v2.0)

```
BNB Chain (L1 — Planned Settlement Anchor)
       |
       | Bridge (not yet built)
       |
Vyniq Chain (L2 — Execution, Consensus, SocialFi)
       |
       +-- API Server (TypeScript): REST API, JSON-RPC, WebSocket, Social APIs
       +-- L2 Engine (TypeScript): Sequencer, batch proofs, state management
       +-- Validator Node (Rust): PoW / PoA consensus, P2P gossip
       +-- P2P Network (Rust/libp2p): gossipsub mesh, peer discovery
       +-- SocialFi Platform (TypeScript): Profiles, posts, communities, rewards
       +-- Reputation Engine (TypeScript): On-chain scoring, trust metrics
       +-- Mobile Light Node (Rust): Future lightweight validation
       +-- Mobile Wallet (React Native): Send/receive, SocialFi, staking
```

### 1.2 Component Matrix

| Layer | Component | Language | Role |
|-------|-----------|----------|------|
| L1 | BNB Chain | — | Security & bridge settlement (planned) |
| L2 | API Server | TypeScript | Canonical source, REST API, JSON-RPC, WebSocket, Social APIs |
| L2 | Validator Node | Rust | PoW mining, block validation, P2P integration |
| L2 | P2P Network | Rust | Peer discovery, block gossip (libp2p) |
| L2 | L2 Engine | TypeScript | Sequencer, batch proofs, state management |
| L2 | SocialFi Platform | TypeScript | Profiles, content, communities, rewards |
| L2 | Reputation Engine | TypeScript | Scoring algorithm, Sybil resistance |
| L2 | Mobile Light Node | Rust | Mobile-friendly read-only client (future) |

### 1.3 Core Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Signature Scheme | Ed25519 (RFC 8032) | Faster than ECDSA; smaller signatures; mobile-efficient |
| Consensus (Phase 1) | PoW (difficulty-targeted) | Simple bootstrap mechanism |
| Consensus (Planned) | Proof-of-Authority | Target for future transition |
| P2P Stack | libp2p (gossipsub, Noise, Yamux) | Battle-tested, modular, mobile-compatible |
| L1 Anchor (Planned) | BNB Chain | Fast finality, low cost, large ecosystem |
| Block Time | 2 seconds (target) | Prototype achieves ~2s on single node |
| Fee Model | 50% burn, 50% to validators | Deflationary pressure |
| Storage (Current) | JSON files | Development speed and auditability |
| Social Identity | Ed25519 wallet-based | Every wallet is a social profile |
| Content Storage | On-chain references + IPFS (planned) | Decentralized content hosting |

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
  "socialfiUrl": "https://social.vyniq.xyz",
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
  "result": "VyniqChain/4201/v2.0.0"
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
  "latestBlock": "2602...",
  "socialfiProfiles": 42,
  "socialfiPosts": 156
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
  "profiles": 8,
  "communities": 3,
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
| `eth_subscribe` | `"socialfi_feed"` | Receive new SocialFi posts and interactions |
| `eth_subscribe` | `"socialfi_rewards"` | Receive reward distribution events |
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

**Server → Client (SocialFi feed update):**
```json
{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "subscription": "0x2",
    "result": {
      "type": "post",
      "postId": "post_abc123",
      "author": "a1b2c3d4e5f6...",
      "content": "Hello Vyniq SocialFi!",
      "timestamp": "0x1781086700"
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
| `SocialAction` | `/vyniq/social/1.0.0` | SocialFi action broadcast |

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
| `socialAction` | Gossipsub social message | Process social action |
| `peerConnected` | New peer connection | Track in peer list |
| `peerDisconnected` | Peer disconnects | Remove from peer list |
| `syncComplete` | Chain sync finished | Log sync results |

### 7.4 Message Flow

**Transaction Propagation:**
1. User submits tx via `POST /transaction/signed`
2. API validates, adds to mempool
3. P2P broadcasts to `vyniq-transactions` topic
4. All connected peers receive and add to their mempools

**SocialFi Action Propagation:**
1. User performs social action (post, like, tip)
2. API validates social action
3. P2P broadcasts to `vyniq-social` topic
4. Peers update their SocialFi state

**Block Propagation:**
1. Miner calls `POST /mine`
2. Block is created and added to local chain
3. P2P broadcasts to `vyniq-blocks` topic
4. Peers validate and append to their chains

---

## 8. L2 Sequencer Engine

The L2 sequencer (`l2-engine/`, ~600+ lines TypeScript) handles:
- Batch production at 3-second intervals
- State management and state root computation
- Bridge event tracking (proof-of-concept, not production-ready)
- SocialFi state integration

Key characteristics:
- **Batch interval:** 3 seconds
- **State persistence:** JSON files
- **Validation pipeline:** Transaction signature verification, nonce checks, balance checks, deduplication
- **SocialFi state:** Social profiles, posts, and reputation tracked in batch state

---

## 9. SocialFi Architecture

### 9.1 Overview

The SocialFi platform is a decentralized social networking layer built natively on Vyniq Chain. Every wallet address can optionally link to a social profile, enabling content creation, community participation, and reputation building.

### 9.2 Architecture Diagram

```
SocialFi Platform (Layer-2 Native on Vyniq Chain)
    |
    +-- User Profiles — On-chain identity linked to wallet address
    +-- Content Layer — Posts, media, comments with token-gating
    +-- Communities — Token-gated groups with governance
    +-- Reputation Engine — Algorithmic scoring from on-chain activity
    +-- Reward System — VYN token incentives for contributions
    +-- Mobile Wallet Integration — Unified interface
```

### 9.3 Core Components

#### Profile Manager

Manages on-chain user profiles linked to wallet addresses:
- Profile creation (username, avatar, bio)
- Profile updates
- Profile verification (badges, attestations)
- Reputation tracking

#### Content Engine

Handles creation and distribution of social content:
- Text posts with optional media attachments
- Comments and threaded replies
- Token-gated content (VYN or NFT required to view)
- Content discovery and feed generation

#### Community System

Token-gated communities with on-chain governance:
- Community creation (VYN fee)
- Token-based membership criteria
- Community treasury management
- Member governance (proposals, voting)

### 9.4 SocialFi Data Structures

#### SocialProfile

```typescript
interface SocialProfile {
  address: string;
  username: string;
  avatar: string;        // IPFS hash (planned) or URL
  bio: string;
  reputation: ReputationScore;
  followerCount: number;
  followingCount: number;
  createdAt: number;
  updatedAt: number;
}
```

#### SocialPost

```typescript
interface SocialPost {
  postId: string;
  author: string;        // wallet address
  content: string;
  media: string[];       // IPFS hashes (planned)
  communityId?: string;
  tokenGate?: { token: string; amount: number };
  likes: number;
  tips: number;
  comments: Comment[];
  createdAt: number;
}
```

#### Community

```typescript
interface Community {
  communityId: string;
  name: string;
  description: string;
  owner: string;
  membershipToken: string;
  membershipThreshold: number;
  memberCount: number;
  treasury: number;
  createdAt: number;
}
```

---

## 10. User Data Flow

### 10.1 SocialFi Data Flow

```
Mobile User
    |
    v
Mobile Wallet App (React Native)
    |
    +---> Blockchain Ops: Send, Receive, Stake
    +---> SocialFi Ops: Create Profile, Post, Like, Tip, Follow
    |
    v
API Server (TypeScript)
    |
    +---> Transaction Processing
    |       +---> Chain State (blocks, transactions)
    |       +---> Account State (balances, nonces)
    |
    +---> SocialFi Processing
    |       +---> Profile State (profiles, usernames)
    |       +---> Content State (posts, comments)
    |       +---> Community State (membership, treasury)
    |       +---> Reputation State (scores, history)
    |
    v
Data Layer
    |
    +---> data/chain.json
    +---> data/accounts.json
    +---> data/socialfi/profiles.json
    +---> data/socialfi/posts.json
    +---> data/socialfi/communities.json
    +---> data/socialfi/reputation.json
    +---> data/vyn-state.json
    +---> data/mempool.json
```

### 10.2 Data Synchronization

- **Blockchain State** — Consensus-driven, appended via PoW mining
- **SocialFi State** — Updated via validated social actions (posts, tips, etc.)
- **Cross-referencing** — Social transactions reference on-chain wallet state
- **Conflict Resolution** — Last-write-wins for profile updates; append-only for content

---

## 11. Wallet-to-Social Linking

### 11.1 Identity Model

```
Ed25519 Keypair
    |
    v
SHA-256 Public Key -> Address (40 hex chars)
    |
    v
SocialProfile (optional, linked to address)
    |
    +-- username (unique, human-readable)
    +-- avatar (IPFS/URL reference)
    +-- bio (text description)
    +-- reputation (algorithmic score)
```

### 11.2 Linking Mechanism

Every Vyniq Chain wallet can link to a SocialFi profile:

1. User creates wallet (`POST /wallet/create`)
2. User creates social profile (`POST /social/profile/create`)
3. Profile is linked to wallet address (1:1 mapping)
4. Social actions (posts, tips, likes) are signed with the wallet's Ed25519 key
5. Reputation accrues to the address based on social activity

### 11.3 Verification

- **Signature Verification** — Social actions are Ed25519-signed, verifiable on-chain
- **Uniqueness** — One profile per address; verified via signature
- **Portability** — Same identity across wallet, SocialFi, and future dApps

---

## 12. Token Reward Flow

### 12.1 Reward Distribution Pipeline

```
User Social Activity
    |
    v
SocialFi Reward Algorithm
    |
    +-- Quality Scoring (engagement, originality, helpfulness)
    +-- Sybil Resistance (reputation minimum, staking requirement)
    +-- Frequency Capping (per-user daily limits)
    |
    v
Reward Calculation
    |
    +-- Daily Pool Allocation (from Community & Ecosystem allocation)
    +-- Per-Action Weighting (post > comment > like)
    +-- Multiplier (reputation score, staking amount)
    |
    v
VYN Distribution
    |
    +-- VYN tokens transferred to user wallet
    +-- On-chain reward event recorded
    +-- User reputation score updated
```

### 12.2 Reward Types

| Action | Base Reward (VYN) | Max Daily | Weight |
|--------|-------------------|-----------|--------|
| Create Post | 0.5 VYN | 10 posts | 1.0x |
| Comment | 0.25 VYN | 20 comments | 0.5x |
| Like / React | 0.1 VYN | 50 likes | 0.2x |
| Receive Tip | Tip amount × 0.95 | Unlimited | N/A |
| Community Creation | 5 VYN | 1 per week | 10.0x |
| Moderation Action | 1 VYN | 10 actions | 2.0x |

### 12.3 Sybil Resistance

- Minimum 100 VYN staked to earn rewards (Phase 2+)
- Reputation score below threshold gets reduced rewards
- New accounts have 7-day cooldown before earning
- IP-level rate limiting for automated detection

---

## 13. Reputation Engine

### 13.1 Algorithm Overview

The reputation engine computes a trust score for each wallet address based on on-chain activity:

```
Reputation = f(TransactionHistory, SocialActivity, Staking, Verification)
```

### 13.2 Score Components

| Component | Weight | Calculation | Data Source |
|-----------|--------|-------------|-------------|
| Transaction History | 25% | log(age_in_days + 1) × volume_factor | Transaction records |
| Social Contributions | 25% | Post quality × engagement_rate | SocialFi posts |
| Community Participation | 20% | Active communities × participation_depth | Community state |
| Staking & Governance | 15% | Stake_amount × stake_duration | Stake records |
| Verification | 10% | Binary: verified or not | Verification records |
| Peer Endorsements | 5% | Endorsements × endorser_reputation | Endorsement graph |

### 13.3 Sybil Resistance Mechanisms

| Mechanism | Description |
|-----------|-------------|
| **Non-transferable** | Reputation is address-bound; cannot be transferred |
| **Time-weighted** | Older accounts have higher weight ceiling |
| **Decay** | Inactivity > 90 days causes score decay |
| **Graph Analysis** | Future: Sybil-resistant graph clustering |
| **Minimum Stake** | Phase 2+: minimum VYN staked for weighted reputation |

### 13.4 Reputation Scoring Example

```typescript
interface ReputationScore {
  address: string;
  overall: number;           // 0-1000
  transactionScore: number;
  socialScore: number;
  communityScore: number;
  stakingScore: number;
  verificationScore: number;
  endorsementScore: number;
  lastUpdated: number;
  rank: number;              // percentile rank
}
```

---

## 14. Social APIs

### 14.1 Profile Endpoints

#### POST /social/profile/create

Create a new SocialFi profile.

**Request:**
```json
{
  "address": "a1b2c3d4e5f6...",
  "username": "vyniq_user",
  "avatar": "https://ipfs.io/ipfs/QmHash...",
  "bio": "Blockchain enthusiast and creator"
}
```

**Response:**
```json
{
  "success": true,
  "address": "a1b2c3d4e5f6...",
  "username": "vyniq_user",
  "createdAt": 1781242280302
}
```

#### GET /social/profile/:address

Get a SocialFi profile by wallet address.

**Response:**
```json
{
  "address": "a1b2c3d4e5f6...",
  "username": "vyniq_user",
  "avatar": "https://ipfs.io/ipfs/QmHash...",
  "bio": "Blockchain enthusiast and creator",
  "reputation": { "overall": 750, "rank": 12 },
  "followerCount": 45,
  "followingCount": 23,
  "createdAt": 1781242280302
}
```

#### PUT /social/profile/update

Update a SocialFi profile. Requires Ed25519 signature verification.

**Request:**
```json
{
  "address": "a1b2c3d4e5f6...",
  "username": "vyniq_user_updated",
  "bio": "Updated bio",
  "signature": "hex_signature..."
}
```

### 14.2 Content Endpoints

#### POST /social/post/create

Create a new post.

**Request:**
```json
{
  "author": "a1b2c3d4e5f6...",
  "content": "Hello Vyniq SocialFi!",
  "media": [],
  "communityId": null,
  "tokenGate": null,
  "signature": "hex_signature..."
}
```

**Response:**
```json
{
  "success": true,
  "postId": "post_abc123",
  "createdAt": 1781242280302
}
```

#### GET /social/feed

Get the global or community feed.

**Parameters:**

| Param | Type | Description |
|-------|------|-------------|
| communityId | `string` | (Optional) Filter by community |
| limit | `number` | (Optional) Max posts (default 20) |
| offset | `number` | (Optional) Pagination offset |

**Response:**
```json
{
  "posts": [
    {
      "postId": "post_abc123",
      "author": "a1b2c3...",
      "username": "vyniq_user",
      "content": "Hello Vyniq SocialFi!",
      "likes": 5,
      "tips": 10,
      "comments": 2,
      "createdAt": 1781242280302
    }
  ],
  "total": 1,
  "limit": 20,
  "offset": 0
}
```

#### POST /social/post/like

Like a post.

#### POST /social/post/tip

Send a VYN tip on a post.

**Request:**
```json
{
  "from": "a1b2c3...",
  "to": "d4e5f6...",
  "postId": "post_abc123",
  "amount": 10,
  "signature": "hex_signature..."
}
```

### 14.3 Community Endpoints

#### POST /social/community/create

Create a token-gated community.

**Request:**
```json
{
  "owner": "a1b2c3d4e5f6...",
  "name": "Vyniq Creators",
  "description": "A community for creators on Vyniq Chain",
  "membershipToken": "VYN",
  "membershipThreshold": 100,
  "signature": "hex_signature..."
}
```

#### GET /social/community/:id

Get community details.

#### POST /social/community/join

Join a token-gated community. Server verifies token balance.

### 14.4 Reputation Endpoints

#### GET /social/reputation/:address

Get reputation score for an address.

**Response:**
```json
{
  "address": "a1b2c3d4e5f6...",
  "overall": 750,
  "components": {
    "transactionHistory": 200,
    "socialContributions": 190,
    "communityParticipation": 150,
    "stakingAndGovernance": 110,
    "verification": 80,
    "peerEndorsements": 20
  },
  "rank": 12,
  "percentile": 95,
  "lastUpdated": 1781242280302
}
```

### 14.5 Reward Endpoints

#### GET /social/rewards/:address

Get reward history for an address.

---

## 15. Mobile Synchronization

### 15.1 Offline-First Architecture

The Vyniq mobile wallet + SocialFi app implements an offline-first synchronization model:

```
User Action (offline)
    |
    v
Local Queue (device storage)
    |
    +-- Transaction Queue (pending send/receive)
    +-- Social Queue (pending posts, likes, tips)
    |
    v
Connectivity Event
    |
    v
Synchronization
    |
    +-- Submit queued transactions to API Server
    +-- Submit queued social actions to API Server
    +-- Fetch latest chain state
    +-- Fetch latest SocialFi feed
    +-- Update local cache
```

### 15.2 Sync Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| syncInterval | 30 seconds | Interval for background sync |
| queueMaxSize | 100 | Max pending actions before forced sync |
| cacheTTL | 300 seconds | Local cache time-to-live |
| mediaQuality | Medium | Image/video quality for bandwidth saving |
| autoSync | true | Automatic sync on connectivity |

### 15.3 Bandwidth Optimization

- **Delta Sync** — Only fetch new/changed data since last sync
- **Media Compression** — Configurable image quality (Low/Medium/High)
- **Batch Requests** — Multiple actions submitted in single HTTP request
- **Push Notifications** — Real-time alerts without polling
- **Cache-First** — Display cached data while syncing in background

---

## 16. Future AI Moderation

### 16.1 Architecture (Planned)

The AI moderation system will operate as a decentralized service layer:

```
User Content
    |
    v
AI Moderation Service (Decentralized)
    |
    +-- Content Classification (spam, abuse, misinformation)
    +-- Sentiment Analysis
    +-- Image/Video Moderation
    |
    v
Moderation Decision
    |
    +-- Flag for Community Review (borderline cases)
    +-- Auto-Reject (clear violations)
    +-- Auto-Approve (safe content)
    |
    v
On-chain Action
    |
    +-- Content flagged/removed
    +-- Reporter rewarded (VYN)
    +-- Appeal process initiated if disputed
```

### 16.2 Design Principles

- **Privacy-preserving** — Content analyzed without exposing user identity
- **Decentralized** — Multiple AI nodes prevent censorship
- **Community-override** — Human moderators can override AI decisions
- **Transparent** — Moderation decisions recorded on-chain
- **Appealable** — Users can appeal moderation decisions

### 16.3 Implementation Status

| Component | Status |
|-----------|--------|
| AI Moderation Specification | Research phase |
| Content Classification Model | Not started |
| Decentralized Inference | Research phase |
| Appeal Mechanism | Design phase |
| Integration with SocialFi | Planned for 2027+ |

---

## 17. Database Schemas & Persistence

### 17.1 Storage Files

All data is stored as JSON files in the `data/` directory:

| File | Content | Purpose |
|------|---------|---------|
| `data/chain.json` | All blocks | Block storage, chain state |
| `data/accounts.json` | Wallet balances, nonces, history | Account state |
| `data/wallets.json` | Public keys | Wallet index |
| `data/mempool.json` | Pending transactions | Transaction pool |
| `data/vyn-state.json` | Token mint/burn state | VYN token state |
| `data/socialfi/profiles.json` | SocialFi profiles | User profiles |
| `data/socialfi/posts.json` | SocialFi posts | Content storage |
| `data/socialfi/communities.json` | SocialFi communities | Community state |
| `data/socialfi/reputation.json` | Reputation scores | Trust metrics |
| `data/socialfi/rewards.json` | Reward distribution history | Reward tracking |
| `data/server.lock` | Lock file | Prevents duplicate instances |

### 17.2 Data Persistence Notes

- All data reloads automatically on server restart
- Storage is currently JSON-based; SQLite migration is planned
- SocialFi storage uses a subdirectory structure for organization
- No external database dependency required for current operation

---

## 18. Transaction Lifecycle

### 18.1 Flow (v2.0 with SocialFi)

1. User generates Ed25519 keypair (client-side)
2. User creates transaction or social action
3. User signs with private key
4. Signed transaction/social action submitted via REST API or WebSocket
5. API Server validates: address format, signature, nonce, balance, deduplication
6. Validated transaction enters mempool
7. Block is mined (PoW) or proposed (sequencer mode)
8. Block appended to chain with Merkle root and parent hash
9. Block gossiped through P2P network
10. SocialFi state is updated if transaction contained social actions

### 18.2 Validation Rules

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

## 19. Consensus Mechanism

### 19.1 Phase 1: Proof of Work (Current)

- Difficulty-targeted PoW mining
- Configurable difficulty
- ~2 second block time on single node
- SHA-256 based mining
- Single-node setup (current); multi-validator not yet deployed

### 19.2 Phase 2: Proof of Authority (Planned)

- 7–21 authorized validators
- BFT security model (f < n/2)
- Block rotation among validators
- Slashing conditions for misbehavior

### 19.3 Phase 3: Hybrid + Mobile (Research)

- Gradual inclusion of mobile light nodes
- Delegated validation for resource-constrained devices
- Economic finality model

### 19.4 Token Emission Schedule (Current Testnet)

| Property | Value |
|----------|-------|
| Max Supply | 10,000,000,000 VYN |
| Initial Block Reward | 50 VYN per block |
| Halving Interval | Every 210,000 blocks (~4.9 years at 2s block time) |
| Fee Burn Rate | 50% of transaction fees |
| Current Testnet Supply | ~1,470 VYN minted (single node) |

---

## 20. Testnet Deployment Guide

### 20.1 Prerequisites

- Ubuntu 22.04 LTS or later (VPS)
- Node.js 18+
- PM2 process manager
- Nginx (for reverse proxy)
- SSL certificate (Let's Encrypt)
- Domains: `rpc.vyniq.xyz`, `explorer.vyniq.xyz`, `faucet.vyniq.xyz`, `social.vyniq.xyz`

### 20.2 Quick Start

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

### 20.3 DNS Configuration

| Domain | Type | Value |
|--------|------|-------|
| `rpc.vyniq.xyz` | A | `<VPS_IP>` |
| `explorer.vyniq.xyz` | A | `<VPS_IP>` |
| `faucet.vyniq.xyz` | A | `<VPS_IP>` |
| `social.vyniq.xyz` | A | `<VPS_IP>` |

### 20.4 Security

```bash
sudo ufw allow 22/tcp && sudo ufw allow 80/tcp && sudo ufw allow 443/tcp && sudo ufw enable
sudo apt install -y fail2ban
```

### 20.5 Multi-Node Network

**Node 1 (bootstrap):**
```json
{ "nodeName": "validator-1", "p2pPort": 9837, "apiPort": 3000 }
```

**Node 2 (connects to Node 1):**
```json
{ "nodeName": "validator-2", "p2pPort": 9838, "apiPort": 3001,
  "bootstrapPeers": ["/ip4/127.0.0.1/tcp/9837/p2p/PeerId1"] }
```

### 20.6 Verification Checklist

- Server starts without errors
- RPC responds on port 3000
- SSL certificate valid
- `eth_chainId` returns `0x1069`
- `eth_blockNumber` returns current height
- `eth_getBalance` returns correct balance
- Transaction submission via `eth_sendRawTransaction`
- SocialFi API endpoints respond
- Profile creation and query work
- Explorer loads and shows data
- SocialFi feed displays content
- Wallet app connects via RPC
- `GET /rpc/health` returns 200

---

## 21. Data Structures

### 21.1 Block

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

### 21.2 Transaction

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

### 21.3 VynAccount

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

### 21.4 SocialProfile

```typescript
interface SocialProfile {
  address: string;
  username: string;
  avatar: string;
  bio: string;
  reputation: ReputationScore;
  followerCount: number;
  followingCount: number;
  createdAt: number;
  updatedAt: number;
}
```

### 21.5 SocialPost

```typescript
interface SocialPost {
  postId: string;
  author: string;
  content: string;
  media: string[];
  communityId?: string;
  tokenGate?: { token: string; amount: number };
  likes: number;
  tips: number;
  comments: Comment[];
  createdAt: number;
}
```

### 21.6 Community

```typescript
interface Community {
  communityId: string;
  name: string;
  description: string;
  owner: string;
  membershipToken: string;
  membershipThreshold: number;
  memberCount: number;
  treasury: number;
  createdAt: number;
}
```

### 21.7 ReputationScore

```typescript
interface ReputationScore {
  address: string;
  overall: number;
  transactionScore: number;
  socialScore: number;
  communityScore: number;
  stakingScore: number;
  verificationScore: number;
  endorsementScore: number;
  lastUpdated: number;
  rank: number;
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
| SocialFi Platform | `socialfi-platform/` (planned) | TBD | TypeScript |
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
| SocialFi Endpoint | `https://social.vyniq.xyz` (planned) |
| Chain ID | 4201 |
| Block Time | ~2 seconds |
| Transaction Throughput (single node) | ~500 tps (untested under load) |
| Validators | 1 (single-node) |
| Faucet | 10 VYN/claim, 24h cooldown |
| Wallet Compatibility | MetaMask (via JSON-RPC) |
| SocialFi Status | In development |
| License | MIT |
