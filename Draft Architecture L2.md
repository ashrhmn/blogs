## Cirkle Blockchain Architecture

```
                                              ┌─────────────┐
                                              │  Explorer   │
                                              │  (React +   │
                                              │   Go API)   │
                                              └─────▲───────┘
                                                    │ JSON-RPC
                                                    │
┌────────────┐       libp2p Gossipsub       ┌───────────────┐
│            │ ◄──────────────────────────► │               │
│  CLI Wallet│      libp2p PubSub &         │  Core Node    │
│  (Go CLI)  │       Peer Discovery         │   (Go)        │
│            │                              │               │
└────────────┘                              └───────────────┘
       ▲                                            ▲
       │                                            │
       │                                ┌───────────┴───────────┐
       │                                │   Consensus Engine    │
       │                                │   (PoS Validator      │
       │                                │    Selection + BFT)   │
       │                                └───────────▲───────────┘
       │                                            │ State
       │                                            │ Transition
       │                                            │
       │   JSON-RPC / gRPC              ┌───────────┴───────────┐
       └────────────────────────────────│    State Machine      │
                                        │  (EVM Engine + Merkle │
                                        │       Trie)           │
                                        └───────────▲───────────┘
                                                    │
                                                    │
                                         ┌──────────┴───────────┐
                                         │      Mempool         │
                                         │ (TX Pool + Validator │
                                         │   Block Assembler)   │
                                         └──────────▲───────────┘
                                                    │ Blocks & TXs
                                                    │
                                          libp2p    │
                                          PubSub    │
                                                    │
┌───────────────┐                              ┌────┴────┐
│  Relayer /    │                              │  Bridge │
│ Monitoring    │                              │ Module  │
│ & Metrics     │                              │ (Go)    │
│ (Prom/Graf)   │                              └─────────┘
└───────────────┘
```

---

### Components

Components 1,2,3,4 are required for an MVP

1. **P2P Layer (libp2p)**

   - **Peer Discovery** via mDNS/DHT
   - **Gossipsub** for:

     - Transaction propagation
     - Block propagation

   - **Secure Channels** (Noise/TLS)

2. **Core Node (Go)**

   - **Consensus Engine**

     - PoS validator selection
     - BFT finality

   - **State Machine**

     - EVM-compatible execution (go-ethereum’s `evm` package)
     - Merkle trie for world state

   - **Mempool & Block Assembler**

     - Transaction validation, gas accounting
     - Block creation by elected validator

   - **APIs**

     - JSON-RPC & gRPC for wallets, explorer, and relayer
     - REST for health checks

3. **Wallet (Go CLI & MetaMask-compatible)**

   - Key management (BIP-39, secp256k1)
   - Transaction creation & signing
   - Peer discovery for local devnet (optional via libp2p)
   - RPC calls to Core Node for balance & tx submission

4. **Bridge Module (Go)**

   - **Relayer Service** listens to:

     - Ethereum deposit events (via web3)
     - Cirkle withdrawal events (via libp2p PubSub or RPC)

   - Initiates cross-chain mint/burn transactions

5. **Explorer (React + Go API)**

   - **Indexer Service** (Go) subscribes to libp2p PubSub or polls JSON-RPC
   - Stores blocks/txs in database
   - Exposes REST/GraphQL for frontend
   - Real-time updates via WebSocket

6. **Monitoring & Metrics**

   - **Prometheus** exporter in Core Node and Explorer
   - **Grafana** dashboards for:

     - Peer count, mempool size, block times
     - Validator performance & uptime

---

#### Data Flows

- **Transaction Flow:**
  Wallet ▶ JSON-RPC ▶ Core Node ▶ libp2p PubSub ▶ All Peers ▶ Mempool ▶ Block Assembler ▶ Block PubSub ▶ All Peers ▶ State Machine

- **Block Propagation:**
  Core Node (Validator) ▶ libp2p Gossipsub ▶ Other Nodes ▶ Block Validation ▶ State Update

- **Explorer Indexing:**
  Explorer Indexer ▶ (libp2p subscription or RPC polling) ▶ Processes Blocks/Txs ▶ Database ▶ API ▶ Frontend

- **Cross-Chain Bridge:**
  Relayer ▶ Ethereum ▶ Bridge Contract ▶ Events ▶ Cirkle ▶ libp2p or RPC ▶ Mint/Burn on Cirkle

---
