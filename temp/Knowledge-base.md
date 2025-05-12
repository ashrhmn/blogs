# Krown L2 – Reference  
*Post-Quantum ZK-Roll-up secured on L1*

---

## 1. High-Level Design

| Layer | Purpose | Highlights |
|-------|---------|------------|
| **Krown L2 Roll-up** | Fast UX, EVM contracts | • Blocks ≈ 2 s via PoS + BFT • off-chain **Sequencer** orders & batches txs <br> • ZK validity proof for each checkpoint |
| **L1** | Security & settlement | • `RollupInbox` stores `{batchHash, stateRoot, proof}` <br>• Escrow & exit processing |
| **Data-Availability** | Permanent calldata storage | • **v1** → post calldata in the same L1 tx <br>• **v2** → Celestia/EigenDA with hash anchored on-chain |

---

## 2. Cryptography Stack

| Primitive | Replaces | Used for |
|-----------|----------|----------|
| **Dilithium** | ECDSA | Sign txs & block headers |
| **Kyber-KEM** | ECDH / Noise | Encrypt libp2p streams |
| **SHAKE-256** | Keccak-256 | All Merkle & state-trie hashes |
| **SPHINCS+** | Plasma exits | Quantum-safe withdrawal proofs |

---

## 3. Node Roles & Off-chain Services

| Component | Runs where? | Responsibilities |
|-----------|-------------|------------------|
| **Sequencer** | Off-chain daemon | Pull top-fee txs → order → batch → hand to validator; trigger ZK prover; post checkpoints to L1 |
| **Consensus Client** | Validator nodes | libp2p gossip, Dilithium verify, mempool, PoS + BFT voting, block propagation |
| **Execution Client** | Full nodes | Re-execute txs, update SHAKE-256 state trie, emit `stateRoot` |
| **Off-chain ZK Prover** | Prover cluster | Build SNARK/STARK for each batch, output succinct proof |
| **RollupInbox** | Ethereum contract | Verify proofs, store roots, unlock / lock assets |
| **Bridge Relayer** | Off-chain daemon | Submit checkpoint tx; relay deposit events L1 → L2 |
| **Explorer / Indexer** | Off-chain service | Ingest blocks, expose REST/GraphQL, UI dashboards |

> **Why off-chain Sequencer?** sub-200 ms inclusion, tiny consensus messages, cheap proof batching.  
> **Why off-chain Prover?** SNARK generation is CPU/GPU-heavy; only the 1 kB proof goes on-chain.

---

## 4. Transaction Lifecycle (step-by-step)

1. **Creation** – Wallet RLP-encodes fields → SHAKE-256 hash → Dilithium sign → broadcast via libp2p.  
2. **Validation** – Peers verify sig / balance / nonce → add to mempool.  
3. **Sequencing & Batch Assembly**  
    ```pseudo
   txList ← mempool.popByFee(maxGas)
   batch  ← txList
   pendingBatch += batch
    ```

4. **Block Assembly (Validator)** – compute `txRoot`, preview `stateRoot`, Dilithium-sign header.
5. **Propagation & Execution** – Peers verify header, replay txs, recompute `stateRoot`.
6. **Finality** – PoS + BFT ≥ ⅔ stake votes → block immutable.
7. **Checkpoint Trigger** – every N blocks / M bytes / T s → Sequencer invokes **ZK Prover**.
8. **L1 Verification** – Sequencer submits `{batchHash, stateRoot, proof}`; RollupInbox verifies; batch final after \~ 12 L1 confirmations.
9. **Withdrawals (L2→L1)** – Exit tx + SPHINCS+ proof → L1 bridge releases funds.
10. **Deposits (L1→L2)** – User locks in bridge; event relayed; L2 credits wallet.

---

## 5. Data Availability – Why Post Calldata?

A ZK proof proves correctness **but not contents**.
Publishing calldata lets anyone:

* Rebuild state from genesis.
* Generate fresh proofs.
* Fork the chain if Sequencer misbehaves.

**v1** post calldata on Ethereum (max security).<br> **v2** switch to Celestia/EigenDA (cheaper) while anchoring the hash on-chain.

---

## 6. Block Header Anatomy

| Field         | Role                           |
| ------------- | ------------------------------ |
| `parentHash`  | link to previous block         |
| `height`      | monotonic counter              |
| `txRoot`      | MerkleSHAKE256(commit tx list) |
| `stateRoot`   | TrieSHAKE256(post-state)       |
| `gasUsed`     | economics / limit tracking     |
| `sequencerId` | identifies ordering authority  |
| `signature`   | Dilithium authenticates header |

---

## 7. FAQ

| Q                                 | A                                                                                                        |
| --------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Checkpoint?**                   | Default 20 blocks **or** 60 s **or** 120 kB; governance can tune.                                        |
| **Sequencer withholds data?**     | RollupInbox rejects next proof; governance slashes Sequencer and rotates key.                            |
| **Is priority just highest fee?** | Yes by default; can switch to fair-ordering or QoS lanes via config.                                     |
| **Similarity to Polygon PoS?**    | Same PoS + BFT skeleton, but Krown adds an external Sequencer, ZK proofs, and post-quantum crypto stack. |

---

## 8. Minimal Sequencer pseudo-code

```pseudo
loop every BLOCK_TIME:
    txs   = mempool.pop(maxGas)
    block = buildBlock(txs)          # txRoot & preview stateRoot
    gossip(block)

    pending += txs
    if checkpointNeeded():
        proof = ZKProver.generate(pending)
        RollupInbox.submit(hash(pending), block.stateRoot, proof)
        pending.clear()
```

---

## 9. External Sequencers – How Common Are They?

| Roll-up / L2         | Sequencer Model                                   | Notes                                                                           |
| -------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Arbitrum One**     | *Single* off-chain sequencer run by Offchain Labs | Provides fast ordering; fall-back trustless “AnyTrust” mode if it goes offline. |
| **Optimism Mainnet** | *Single* external sequencer (Optimism PBC)        | Plans to decentralise with the forthcoming “Cannon” fault-proof stack.          |
| **StarkNet**         | *Single* sequencer operated by StarkWare          | Sequencer batches Cairo txs, hands them to an off-chain STARK prover.           |
| **zkSync Era**       | *Single* external sequencer run by Matter Labs    | Batcher → off-chain SNARK prover → Ethereum verifier.                           |
| **Scroll**           | External sequencer (community-operated roadmap)   | Initial mainnet has centralised operator while proof system is optimised.       |

> **Pattern:** Every production roll-up today begins with **one external sequencer** for UX and operational simplicity, then introduces a rotating set or L1-elected committee later.

---

## 10. Off-chain Provers

| Chain / Roll-up                                                     | Off-chain Prover?                                                                                           | Details                                                                                                           |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Polygon PoS**                                                     | **No**                                                                                                      | It relies on validator honesty and a checkpoint Merkle root; no ZK/Fraud proofs.                                  |
| **Polygon zkEVM**                                                   | **Yes**                                                                                                     | A separate prover cluster (Groth16 → Plonky2 roadmap) generates validity proofs; only the proof is sent on-chain. |
| **All ZK roll-ups (StarkNet, zkSync, Scroll, Polygon zkEVM, etc.)** | **Yes**                                                                                                     | Proof generation is CPU/GPU-intensive, so it always runs off-chain.                                               |
| **Optimistic roll-ups**                                             | *No validity proof*. They rely on on-chain fraud-proofs (also computed off-chain but only when challenged). |                                                                                                                   |

---

#### Why external sequencers & provers are typical

* **Latency** – one fast operator can give users sub-second confirmation, whereas BFT rounds across many validators take longer.
* **Gas efficiency** – batching and proof generation off-chain lets the roll-up post one compact commitment instead of every tx.
* **Engineering boot-strap** – centralised first, then gradually decentralise once economics and tooling mature (e.g., Arbitrum’s upcoming “permissionless” sequencers).

---

## 11. Roadmap Overview

| Phase                  | Key Milestones                                |
| ---------------------- | --------------------------------------------- |
| **Foundations**        | Final spec, PQC libs, P2P prototype           |
| **Core Node**          | PoS + BFT consensus, SHAKE-256 trie execution |
| **Sequencer & ZK**     | Batcher, SNARK circuits, RollupInbox contract |
| **Explorer & Wallet**  | Go indexer, React UI, CLI wallet (Dilithium)  |
| **Token & Incentives** | KROWN economics, faucet, staking rewards      |
| **Testnet**            | Full audit, public faucet, Grafana dashboards |
| **Mainnet**            | Genesis, validator onboarding, DAO governance |

