# Vyniq Chain

A BNB Chain appchain powering the VYN token ecosystem.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    BNB Chain (Base)                      │
│  (Security, finality, bridge anchor for Vyniq Chain)    │
└────────────────────┬────────────────────────────────────┘
                     │  Bridge / Oracle
┌────────────────────▼────────────────────────────────────┐
│                Vyniq Chain (Appchain L2)                 │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │            API Server (Canonical Source)           │   │
│  │  • chain.json — persistent block storage          │   │
│  │  • vyn-state.json — token mint/burn state         │   │
│  │  • accounts.json — wallet balances & staking       │   │
│  │  • mempool.json — pending transaction pool        │   │
│  │  REST API :3000                                    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐ │
│  │  Validator    │  │  P2P Network  │  │  Light Node   │ │
│  │  Node (Rust)  │◄─┤  (libp2p)    │─►│  (Mobile)     │ │
│  │  PoW Mining   │  │  Gossipsub   │  │               │ │
│  └──────────────┘  └──────────────┘  └───────────────┘ │
│                                                          │
│  Consensus: PoW (temporary, locked)                      │
│  Token:     VYN — 10,000,000,000 max supply              │
└──────────────────────────────────────────────────────────┘
```

| Layer  | Component       | Language   | Role                              |
|--------|-----------------|------------|-----------------------------------|
| Base   | BNB Chain       | —          | Security & bridge settlement      |
| L2     | API Server      | TypeScript | Canonical source, REST API        |
| L2     | Validator Node  | Rust       | PoW mining, block validation      |
| L2     | P2P Network     | Rust       | Peer discovery, block gossip      |
| L2     | Light Node      | Rust       | Mobile-friendly read-only client  |

## Token — VYN

| Property       | Value                |
|----------------|----------------------|
| Max Supply     | 10,000,000,000 VYN   |
| Initial Reward | 50 VYN per block     |
| Halving        | Every 210,000 blocks |
| Burn Rate      | 50% of tx fees       |

## Getting Started

### Prerequisites

- [Rust](https://rustup.rs/) (edition 2021)
- [Node.js](https://nodejs.org/) >= 18

### Build all crates

```bash
cargo build --workspace
```

### API server

```bash
npm install
npm run dev:api
```

## License

MIT
