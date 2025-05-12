
# Krown L2 Reference  
*(Post-Quantum ZK-Rollup on L1)*  

---

## 1. High-Level Design

| Layer | Purpose | Highlights |
|-------|---------|------------|
| **Krown L2 Roll-up** | Fast user UX, EVM smart-contracts | • Blocks every ≈ 2 s via PoS + BFT<br>• Off-chain **Sequencer** orders & batches txs<br>• ZK validity proofs for every checkpoint |
| **L1** | Security & settlement | • `RollupInbox` contract stores `{batchHash, stateRoot, proof}`<br>• Funds escrow & exit processing |
| **Data-Availability (DA)** | Permanent calldata storage | • v1 → post calldata directly on Ethereum<br>• v2 → optional Celestia/EigenDA with on-chain hash |

---

## 2. Cryptography Stack

| Primitive | Replaces | Used for |
|-----------|----------|----------|
| **Dilithium** | ECDSA | Sign txs & block headers |
| **Kyber-KEM** | ECDH | Encrypt libp2p channels |
| **SHAKE-256** | Keccak-256 | All Merkle & trie hashes |
| **SPHINCS+** | Plasma exits | Quantum-safe withdrawal proofs |

---

## 3. Node Roles

| Client | Responsibilities |
|--------|------------------|
| **Sequencer (off-chain)** | Pull high-priority txs → order → batch → hand to validator; trigger ZK prover; post checkpoints to L1 |
| **Consensus Client** | libp2p gossip, Dilithium sig verification, mempool, PoS + BFT voting, block propagation |
| **Execution Client** | Re-execute txs, update SHAKE-256 state trie, generate stateRoot |

> **Why keep Sequencer off-chain?**  
> • Sub-200 ms UX & flexible MEV policy.  
> • Consensus traffic stays small (sign header only).  
> • Proof batching window cuts L1 gas 10-100×.

---

## 4. Transaction Lifecycle (step-by-step)

1. **Creation** – Wallet hashes tx with SHAKE-256, signs with Dilithium, broadcasts via libp2p.  
2. **Validation** – Peers verify sig, nonce, balance; push into mempool.  
3. **Sequencing & Batch Assembly**  
    ```pseudo
   txList ← mempool.topByFee()
   batch  ← txList[0 … gasLimit]
   pendingBatch += batch
    ```

4. **Block Assembly (Validator)**
   *Compute `txRoot`, preview `stateRoot`, sign header (Dilithium).*
5. **Propagation & Execution** – Peers verify header, replay txs, recompute stateRoot.
6. **Finality** – PoS + BFT votes ≥ ⅔ stake → block immutable.
7. **Checkpoint Trigger** – Every N blocks / M bytes / T s →
   *Sequencer runs SNARK prover → posts `{batchHash, stateRoot, proof}` to `RollupInbox`.*
8. **L1 Verification** – On-chain verifier accepts proof; batch is final when L1 tx has \~12 confirmations.
9. **Withdrawals** – User exit tx included in batch; `SPHINCS+` leaf proof lets L1 bridge release funds.
10. **Deposits** – L1 bridge event relayed → L2 credits wallet.

---

## 5. Data Availability - Why?

> **Q:** *We already post a ZK proof—why bother uploading calldata?*
> **A:** A proof shows the transition is valid, but doesn’t reveal **what** happened.
> Anyone must be able to reconstruct state, generate new proofs, or fork the chain without trusting the sequencer. Hence every byte of calldata is published—either directly on Ethereum (max-security) or to a DA layer (cheaper, hash anchored on-chain).

---

## 6. Block Header Anatomy

| Field         | Source                   | Role               |
| ------------- | ------------------------ | ------------------ |
| `parentHash`  | previous header          | chain linkage      |
| `height`      | ++                       | monotonic          |
| `txRoot`      | MerkleSHAKE256(tx list)  | input commitment   |
| `stateRoot`   | TrieSHAKE256(post-state) | output commitment  |
| `gasUsed`     | EVM exec                 | block economics    |
| `sequencerId` | ID                       | ordering authority |
| `signature`   | Dilithium                | auth header        |

---

## 7. FAQ

| Question                                          | Answer                                                                                                                                             |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **How often are checkpoints?**                    | Configurable; typical 20 blocks or 60 s or 120 kB—trade-off gas vs. L1 latency.                                                                    |
| **What if Sequencer withholds data?**             | L1 contract refuses future proofs without data-availability hash; community can slash sequencer stake and elect a backup.                          |
| **Polygon similarity?**                           | Both use PoS + BFT and checkpoint to Ethereum, but Krown adds a stand-alone off-chain Sequencer, ZK validity proofs, and full post-quantum crypto. |

---

## 8. Minimal Sequencer Pseudocode (reference)

```pseudo
loop every BLOCK_TIME:
    txs ← mempool.pop(maxGas)
    block ← buildBlock(txs)
    broadcast(block)

    pending += txs
    if checkpointNeeded():
        proof ← ZKProver(pending)
        RollupInbox.submit(batchHash(pending), stateRoot(block), proof)
        pending.clear()
```

---

## 9. Roadmap 

| Phase                  | Milestones                                            |
| ---------------------- | ----------------------------------------------------- |
| **Foundations**        | finalize spec, PQC library integration, P2P prototype |
| **Core Node**          | PoS + BFT consensus, SHAKE-256 trie execution         |
| **Sequencer & ZK**     | batcher service, SNARK circuits, RollupInbox contract |
| **Explorer & Wallet**  | Go indexer, React UI, CLI wallet (Dilithium keys)     |
| **Token & Incentives** | KROWN economics, faucet, staking rewards              |
| **Testnet**            | full audit, public faucet, Grafana dashboards         |
| **Mainnet**            | genesis, validator onboarding, DAO governance         |


