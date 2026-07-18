<div align="center">

<img src="docs/favicon.svg" alt="Vyniq" width="100" />

# Vyniq

### The Web3 Ecosystem for Everyone

**A high-performance blockchain platform with integrated SocialFi, DeFi, and decentralized identity — accessible from any device, anywhere.**

---

[![Version](https://img.shields.io/badge/Version-0.1.0-blue)](https://github.com/mib316127-bit/vyniq-chain)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Testnet%20Active-yellow)](https://docs.vyniq.xyz)
[![Consensus](https://img.shields.io/badge/Consensus-PoA-purple)](https://docs.vyniq.xyz)

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Rust](https://img.shields.io/badge/Rust-2021-000000?logo=rust&logoColor=white)](https://www.rust-lang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![React Native](https://img.shields.io/badge/React_Native-Mobile-61DAFB?logo=react&logoColor=white)](https://reactnative.dev/)

[![Documentation](https://img.shields.io/badge/Documentation-docs.vyniq.xyz-blue?logo=readthedocs&logoColor=white)](https://docs.vyniq.xyz)
[![Twitter/X](https://img.shields.io/badge/Twitter-@Vyniqtechnology-1DA1F2?logo=x&logoColor=white)](https://x.com/Vyniqtechnology)
[![Telegram](https://img.shields.io/badge/Telegram-vyniqlab-26A5E4?logo=telegram&logoColor=white)](https://t.me/vyniqlab)
[![GitHub](https://img.shields.io/badge/GitHub-vyniq--chain-181717?logo=github&logoColor=white)](https://github.com/mib316127-bit/vyniq-chain)

</div>

---

## About

**Vyniq** is a next-generation Web3 ecosystem built to make blockchain technology genuinely accessible. It combines a high-performance blockchain with integrated SocialFi, DeFi, and decentralized identity — all designed to work seamlessly on any device.

At its core, Vyniq is powered by **Proof of Authority (PoA)** consensus, delivering near-instant finality (~2 second blocks) with minimal resource requirements. Transaction fees are distributed **50% burned / 50% to validators**, creating deflationary pressure on the fixed total supply of **10,000,000,000 VYN**.

Vyniq is more than a chain — it's a complete ecosystem comprising a blockchain core, a SocialFi platform with creator economics, a built-in decentralized exchange, a token launchpad, a mobile wallet, and a decentralized identity system called **Wallet ID**.

> **Status:** Testnet operational. SocialFi platform, DEX, and mobile wallet in active development.

---

## Mission

To build the infrastructure that brings the next billion users into Web3 — not by simplifying blockchain, but by rethinking it from the ground up for the devices and networks people actually use.

## Vision

A world where a smartphone in Lagos, Jakarta, or São Paulo provides the same access to decentralized finance, digital identity, and creator economies as a desktop workstation in San Francisco. Where participation in the network is a right, not a privilege gated by hardware.

---

## Features

| Feature | Description |
|---------|-------------|
| **Proof of Authority Consensus** | ~2-second block times with authorized validators. Efficient, deterministic, and scalable. |
| **SocialFi Platform** | Decentralized social networking with creator monetization, reputation systems, and community governance. |
| **Built-in DEX** | Integrated decentralized exchange for native token trading with liquidity pools. |
| **Token Launchpad** | Launch new tokens directly within the ecosystem with built-in fair distribution mechanics. |
| **Wallet ID** | Decentralized identity protocol — one identity across all platforms and services. |
| **Ed25519 Native** | End-to-end Ed25519 signatures across all layers — 3x faster than ECDSA with smaller signatures. |
| **Deflationary Economics** | 50% of all transaction fees permanently burned from supply. |
| **Creator Economy** | Protocol-enforced creator rewards, tipping, and community-driven value distribution. |
| **Cross-Platform** | Mobile wallet (React Native), web apps, API access — build and interact from anywhere. |
| **Open Source** | MIT-licensed codebase. Fork, contribute, and build with the community. |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                     BNB Chain (L1 — Settlement)                   │
│               Security anchor & bridge (future roadmap)           │
└─────────────────────────────┬────────────────────────────────────┘
                              │ Bridge (planned)
┌─────────────────────────────▼────────────────────────────────────┐
│                     Vyniq Ecosystem                               │
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │                  API Server (TypeScript)                    │   │
│  │        REST API · JSON-RPC · WebSocket · Social APIs       │   │
│  └───────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌────────────────┐  ┌────────────────┐  ┌───────────────────┐  │
│  │   Validator     │  │   P2P Network   │  │   L2 Sequencer    │  │
│  │   Node (Rust)   │◄─┤   (libp2p)     │─►│   (TypeScript)    │  │
│  │   PoA Consensus │  │   Gossipsub    │  │   Batch proofs    │  │
│  └────────────────┘  └────────────────┘  └───────────────────┘  │
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │                  SocialFi Platform                          │   │
│  │        Profiles · Communities · Reputation · Rewards        │   │
│  └───────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌────────────────┐  ┌────────────────┐  ┌───────────────────┐  │
│  │  Mobile Wallet   │  │    Explorer     │  │   Community App   │  │
│  │  (React Native)  │  │  (React/Vite)   │  │  (Telegram Mini)  │  │
│  └────────────────┘  └────────────────┘  └───────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

| Layer | Component | Language | Purpose |
|-------|-----------|----------|---------|
| **Settlement** | BNB Chain | — | Security anchor & bridge (future) |
| **Consensus** | Validator Node | Rust | PoA block validation |
| **Networking** | P2P Network | Rust | Gossipsub, Noise encryption, peer discovery |
| **Execution** | L2 Sequencer | TypeScript | Batch production, state management |
| **API** | API Server | TypeScript | REST, JSON-RPC, WebSocket, Social APIs |
| **Social** | SocialFi Platform | TypeScript | Profiles, communities, reputation, rewards |
| **DeFi** | DEX & Launchpad | TypeScript | Token swaps, liquidity, token launches |
| **Client** | Mobile Wallet | React Native | Send/receive, QR, staking, identity |
| **Client** | Block Explorer | React / Vite | Chain analytics and SocialFi insights |
| **Utility** | Community App | Telegram Bot | Mini App for ecosystem engagement |

---

## Technology Stack

<details>
<summary><strong>Core</strong></summary>

| Technology | Version | Purpose |
|------------|---------|---------|
| Rust | 2021 edition | Blockchain core, validator node, P2P networking |
| TypeScript | 5.x | API server, L2 sequencer, SocialFi, DeFi, tooling |
| Node.js | 18+ | Runtime for TypeScript services |
| Ed25519 | RFC 8032 | Digital signatures (end-to-end) |
| libp2p | Latest | P2P networking, gossipsub, Noise encryption |

</details>

<details>
<summary><strong>Frontend & Clients</strong></summary>

| Technology | Purpose |
|------------|---------|
| React Native | Cross-platform mobile wallet |
| React / Vite | Block explorer, documentation site |
| Tailwind CSS | UI styling across all web apps |
| Vanilla JS | Telegram Mini App frontend |

</details>

<details>
<summary><strong>Infrastructure & Tooling</strong></summary>

| Technology | Purpose |
|------------|---------|
| Firebase Admin | Community App backend (Telegram) |
| Grammy | Telegram Bot framework |
| Puppeteer | PDF generation for whitepaper |
| pdf-lib | Programmatic PDF assembly |

</details>

<details>
<summary><strong>Planned</strong></summary>

| Technology | Purpose | Target |
|------------|---------|--------|
| IPFS | Decentralized content storage | 2027+ |
| PostgreSQL | Scalable storage migration | 2026+ |
| zk-Proofs | Privacy-preserving reputation | 2028+ |

</details>

---

## Folder Structure

```
vyniq-chain/
├── api-server/                  # TypeScript API server (REST, JSON-RPC, WebSocket)
│   ├── src/
│   │   ├── dex/                 # DEX modules (swap, liquidity, staking, launchpad)
│   │   ├── p2p/                 # P2P networking (gossipsub, peer discovery)
│   │   ├── config.ts            # Server configuration
│   │   ├── faucet.ts            # Testnet faucet logic
│   │   ├── index.ts             # Entry point
│   │   ├── validator.ts         # Validator client
│   │   └── types.ts             # TypeScript type definitions
│   ├── config/                  # Network & node configuration
│   ├── data/                    # Chain state, accounts, mempool (JSON)
│   └── package.json
│
├── core-chain/                  # Rust core blockchain library
│   └── src/lib.rs               # Transactions, blocks, wallet, chain validation
│
├── p2p-network/                 # Rust P2P networking library
│   └── src/lib.rs               # Gossipsub, Noise encryption, peer discovery
│
├── validator-node/              # Rust validator binary
│   └── src/main.rs              # Block validation with P2P integration
│
├── mobile-light-node/           # Rust mobile light node (stub)
│   └── src/main.rs              # Future lightweight validation client
│
├── l2-engine/                   # TypeScript L2 sequencer
│   ├── src/
│   │   ├── core/                # Chain types and logic
│   │   ├── sequencer/           # Batch production
│   │   ├── state/               # State management
│   │   ├── storage/             # Persistence layer
│   │   ├── rpc/                 # RPC server
│   │   ├── bridge/              # BNB Chain bridge (planned)
│   │   └── crypto/              # Ed25519 signer
│   └── test/                    # Integration tests
│
├── validator-server/            # TypeScript validator HTTP server
│   └── src/index.ts
│
├── faucet/                      # TypeScript testnet faucet
│   └── src/index.ts
│
├── apps/
│   ├── vyniq-mining/            # Telegram Mini App (SocialFi + engagement)
│   │   ├── src/                 # Express + Grammy + Firebase backend
│   │   ├── public/              # Frontend (vanilla JS SPA)
│   │   └── tsconfig.json
│   │
│   ├── vyniq-explorer/          # Block explorer (React + Vite + Tailwind)
│   │   └── src/
│   │       ├── pages/           # Dashboard, blocks, transactions, DEX, staking
│   │       ├── components/      # Reusable UI components
│   │       └── services/        # API client, blockchain utils
│   │
│   └── vyniq-wallet/            # Mobile wallet (React Native)
│       └── src/
│           ├── screens/         # Balance, send, receive, settings
│           ├── components/      # QR code, network switcher
│           └── services/        # Wallet, API, storage
│
├── docs/                        # Documentation website
├── Documentation_V2/            # Technical documentation v2
├── charts/                      # Diagrams and infographics
├── WHITEPAPER_V2.md             # Protocol whitepaper
├── TECHNICAL_DOCS_V2.md         # Technical documentation
├── Cargo.toml                   # Rust workspace root
├── package.json                 # Node.js workspace root
└── tsconfig.json                # TypeScript base config
```

---

## Installation

### Prerequisites

- [Rust](https://rustup.rs/) (edition 2021)
- [Node.js](https://nodejs.org/) >= 18
- [npm](https://www.npmjs.com/) (comes with Node.js)

### Clone & Setup

```bash
# Clone the repository
git clone https://github.com/mib316127-bit/vyniq-chain.git
cd vyniq-chain

# Build Rust crates
cargo build --workspace

# Install JavaScript dependencies
npm install
```

---

## Running Locally

### API Server

```bash
npm run dev:api
```

Starts on `http://localhost:3000` with:

| Endpoint | URL |
|----------|-----|
| REST API | `http://localhost:3000` |
| JSON-RPC | `http://localhost:3000/rpc` |
| WebSocket | `ws://localhost:3000/ws` |

### Validator Node

```bash
npm run dev:validator
```

### L2 Sequencer

```bash
cd l2-engine
npm run dev
```

### Community App (Telegram Mini App)

```bash
npm run dev:mining
```

### Run Tests

```bash
node e2e-full.js
```

---

## API Reference

<details>
<summary><strong>REST Endpoints</strong></summary>

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/status` | Network status, block height, peer count |
| `GET` | `/chain` | Full chain data |
| `GET` | `/block/:hash` | Block details by hash |
| `GET` | `/transaction/:hash` | Transaction details by hash |
| `GET` | `/account/:address` | Account balance and nonce |
| `POST` | `/transaction/signed` | Submit a signed transaction |
| `GET` | `/mempool` | Pending transaction pool |
| `GET` | `/validators` | Active validator set |
| `GET` | `/faucet/:address` | Request testnet VYN |

</details>

<details>
<summary><strong>JSON-RPC Methods</strong></summary>

| Method | Description |
|--------|-------------|
| `eth_blockNumber` | Get latest block number |
| `eth_getBalance` | Get account balance |
| `eth_sendRawTransaction` | Submit signed transaction |
| `eth_getTransactionByHash` | Get transaction details |
| `eth_getBlockByNumber` | Get block by number |

</details>

<details>
<summary><strong>WebSocket</strong></summary>

Real-time subscriptions for new blocks, transactions, and account state changes.

</details>

---

## Smart Contract Development

Vyniq supports **EVM-compatible smart contract development** on its current architecture. Developers can deploy and interact with Solidity contracts via the JSON-RPC API.

**Current capabilities:**
- Native token transfers (VYN)
- Account-based state management
- EVM-compatible smart contract execution (Solidity)
- Contract deployment and interaction via JSON-RPC

**Future roadmap:**
- Native smart contract runtime (custom VM)
- Developer SDK (TypeScript and Rust)
- Advanced contract tooling and IDE integration

> See the [Roadmap](#roadmap) for timeline details.

---

## Documentation

### Whitepaper

The **Vyniq Whitepaper V2.4** is the authoritative source for protocol design, tokenomics, and technical specifications.

- **File:** [WHITEPAPER_V2.md](WHITEPAPER_V2.md)
- **Online:** [docs.vyniq.xyz](https://docs.vyniq.xyz)

Covers: Proof of Authority consensus, 10B VYN tokenomics, 10-category genesis allocation, fee model (50% burn / 50% validator), Wallet ID, governance model, and full protocol specification.

### Technical Documentation

The **Technical Documentation V2.1** provides implementation-level details for developers building on Vyniq.

- **File:** [TECHNICAL_DOCS_V2.md](TECHNICAL_DOCS_V2.md)
- **Online:** [docs.vyniq.xyz](https://docs.vyniq.xyz)

Covers: Architecture deep-dive, block structure, transaction lifecycle, P2P networking, API reference, database schemas, and developer guides.

---

## Tokenomics

**VYN** is the native utility token of the Vyniq ecosystem.

| Spec | Value |
|------|-------|
| **Total Supply** | 10,000,000,000 VYN |
| **Transaction Fee** | Variable per transaction type |
| **Fee Distribution** | 50% burned, 50% to validators |
| **Consensus** | Proof of Authority |
| **Supply Model** | Fixed genesis allocation — no inflation, no block rewards, no mining |

### Allocation

All tokens are pre-allocated at genesis. There are no block rewards, no mining rewards, and no inflationary emission. Validators earn revenue exclusively from transaction fees (50% of fees).

| Category | Allocation |
|----------|------------|
| Foundation Treasury | 20% |
| Investors | 20% |
| Ecosystem Development | 15% |
| Community & Airdrop | 10% |
| Validator Incentives | 10% |
| Liquidity Reserve | 8% |
| Team | 7% |
| Marketing & Growth | 5% |
| Advisors | 3% |
| Strategic Partnerships | 2% |

### Token Utility

| Utility | Description |
|---------|-------------|
| **Gas** | Transaction fees for all network operations |
| **Staking** | Secure the network, earn rewards |
| **Governance** | Vote on protocol parameters |
| **Creator Rewards** | Earn through content creation and engagement |
| **Community Incentives** | Participate, moderate, curate — earn VYN |
| **Tipping** | Direct value transfer between users |
| **Token-Gated Access** | Access exclusive communities and content |

> See [WHITEPAPER_V2.md](WHITEPAPER_V2.md) for full allocation details, vesting schedules, and fee structure.

---

## Roadmap

### Phase 1 — Foundation *(Feb 2026 — Completed)*

- [x] Rust blockchain core — transactions, blocks, wallet, chain validation
- [x] P2P networking — libp2p gossipsub, Noise encryption
- [x] API server — REST, JSON-RPC, WebSocket endpoints
- [x] L2 sequencer — batch production, state management
- [x] Mobile wallet — React Native send/receive, QR, staking UI
- [x] Block explorer — React/Vite with routing and search
- [x] Community App — Telegram Mini App with wallet, tasks, referral system
- [x] Testnet faucet — rate-limited token dispenser
- [x] Testnet launch and public deployment
- [ ] Multi-validator deployment
- [ ] Professional security audit

### Phase 2 — Ecosystem *(Q2 2026 — In Progress)*

- [ ] SocialFi platform — profiles, posts, communities
- [ ] Creator rewards — on-chain engagement-based distribution
- [ ] Reputation engine — portable, Sybil-resistant scoring
- [ ] Token-gated communities
- [ ] Built-in DEX — token swaps and liquidity pools
- [ ] Token launchpad
- [ ] Wallet ID — decentralized identity protocol
- [ ] Developer SDK — TypeScript and Rust
- [ ] Mainnet preparation

### Phase 3 — Growth *(Q3 2026 — In Progress)*

- [ ] Mainnet launch
- [ ] On-chain governance — VYN staker voting
- [ ] BNB Chain bridge
- [ ] Mobile Node Assisted Validation research
- [ ] IPFS integration — decentralized content storage

### Phase 4 — Scale *(2026+ — Vision)*

- [ ] Cross-chain messaging
- [ ] zk-Proofs — privacy-preserving reputation
- [ ] Enterprise solutions — white-label deployments
- [ ] Multi-chain expansion

---

## Ambassador Program

Join the **Vyniq Ambassador Program** and help grow the ecosystem while earning rewards.

**What Ambassadors do:**
- Create educational content about Vyniq
- Represent Vyniq at local and online crypto communities
- Onboard new users and developers
- Provide feedback to the core team

**Ambassador Benefits:**
- VYN token rewards for verified contributions
- Early access to new features and testnets
- Direct line to the core team
- Exclusive Ambassador badge on Wallet ID

> Join the community: [t.me/vyniqlab](https://t.me/vyniqlab)

---

## Contributing

We welcome contributions from the community. Every contribution matters — whether it's a bug fix, new feature, documentation improvement, or feedback.

### How to Contribute

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/my-feature`)
3. **Commit** your changes (`git commit -m 'Add my feature'`)
4. **Push** to the branch (`git push origin feature/my-feature`)
5. **Open** a Pull Request

### Guidelines

- Follow existing code style and conventions
- Write clear commit messages
- Add tests for new functionality
- Update documentation as needed
- Keep PRs focused — one feature or fix per PR

### Areas for Contribution

| Area | Description |
|------|-------------|
| **Core Protocol** | Rust blockchain improvements, consensus research |
| **API Server** | New endpoints, performance optimization |
| **SocialFi** | Creator tools, moderation, reputation algorithms |
| **DeFi** | DEX improvements, launchpad features |
| **Mobile Wallet** | UX improvements, new features |
| **Documentation** | Tutorials, guides, API references |
| **Testing** | Unit tests, integration tests, e2e coverage |

---

## Security

| Measure | Description |
|---------|-------------|
| **Ed25519 Signatures** | End-to-end cryptographic security across all protocol layers |
| **Noise Encryption** | Encrypted P2P communication between validators |
| **Gossipsub** | Authenticated message propagation with peer scoring |
| **Rate Limiting** | API and faucet rate limiting to prevent abuse |
| **HMAC Verification** | Telegram WebApp request authentication |
| **Open Source** | All code is public and auditable under the MIT license |

> **Security Audit:** Professional audit planned before mainnet launch.

If you discover a security vulnerability, please report it responsibly via [GitHub Issues](https://github.com/mib316127-bit/vyniq-chain/issues) or contact the team directly.

---

## License

This project is licensed under the **MIT License**.

```
MIT License

Copyright (c) 2026 Vyniq Technologies

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Contact & Community

<div align="center">

| | Link |
|---|------|
| **Documentation** | [docs.vyniq.xyz](https://docs.vyniq.xyz) |
| **Twitter / X** | [@Vyniqtechnology](https://x.com/Vyniqtechnology) |
| **Telegram** | [t.me/vyniqlab](https://t.me/vyniqlab) |
| **GitHub** | [mib316127-bit/vyniq-chain](https://github.com/mib316127-bit/vyniq-chain) |
| **Whitepaper** | [WHITEPAPER_V2.md](WHITEPAPER_V2.md) |

</div>

---

<div align="center">

**Built with care by [Vyniq Technologies](https://t.me/vyniqlab)**

</div>
