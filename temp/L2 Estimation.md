|      Package                   |      Description                                                                                   | Estimated Time |
| ------------------------------ | ------------------------------------------------------------------------------------------------- | -------------- |
| **Networking Foundation**      | Stand up libp2p transport, Kyber-encrypted hand-shake, peer discovery, and gossip channels.       | **4 weeks**    |
| **Validated Mempool**          | Build fee-priority transaction pool with Dilithium sig checks and spam limits.                    | **3 weeks**    |
| **State-Trie Library**         | Implement SHAKE-256 Merkle-Patricia trie with snapshot / rollback and proof helpers.              | **4 weeks**    |
| **Minimal EVM Execution**      | Integrate core EVM opcodes, gas metering, and post-state diff emitter.                            | **5 weeks**    |
| **PoS Stake Module**           | Add validator registry, bond / unbond CLI, and stake-weighted proposer rotation.                  | **4 weeks**    |
| **BFT Voting Engine**          | Code prevote / precommit rounds, ⅔-stake commit rule, and fork-choice logic.                      | **5 weeks**    |
| **Sequencer Core**             | Off-chain ordering service, batch builder, header pre-sign and broadcast.                         | **4 weeks**    |
| **Block Builder & Gossip**     | Assemble full blocks, enforce gas limit, broadcast on `blocks` topic, enable parallel validation. | **3 weeks**    |
| **ZK Circuit Design**          | Specify public inputs and opcode gadgets; create CI tests for proof / verify.                     | **7 weeks**    |
| **Prover Infrastructure**      | Build witness generator, CPU/GPU worker pool, deterministic Docker image, REST API.               | **5 weeks**    |
| **RollupInbox Contract**       | Deploy L1 contract with `submitBatch`, verifier linkage, and emergency slash hooks.               | **3 weeks**    |
| **Check-point Poster**         | Sequencer sub-module that triggers prover, packs calldata, posts and tracks checkpoints.          | **3 weeks**    |
| **SPHINCS+ Exit Proofs**       | Add quantum-safe leaf signatures and bridge verifier upgrade for withdrawals.                     | **4 weeks**    |
| **ERC-20 Deposit Path**        | Implement L1 `deposit()` lock, relayer listener, and L2 mint handler.                             | **3 weeks**    |
| **ERC-20 Withdrawal Path**     | Build exit tx, Merkle inclusion proof flow, and L1 `withdraw()` unlock.                           | **3 weeks**    |
| **Public Testnet Ops**         | Terraform/Ansible deploy, Prometheus metrics, faucet bot / API for test tokens.                   | **4 weeks**    |
| **CLI Wallet (Dilithium)**     | Command-line key-gen, fee suggestion, Dilithium signing, and RPC queries.                         | **3 weeks**    |
| **Indexer & Explorer v0**      | Block ingestor to Postgres plus minimal REST/GraphQL and React pages.                             | **4 weeks**    |
| **Security Audit & Load-Test** | Independent code audit, k6 stress tests, and remediation of critical issues.                      | **5 weeks**    |
| **Mainnet v1 Launch**          | Run genesis ceremony, deploy HSM-protected Sequencer, enable calldata-on-chain DA.                | **2 weeks**    |
| **External DA Layer**          | Integrate Celestia/EigenDA blob posting, hash anchoring, and fallback switch.                     | **5 weeks**    |
