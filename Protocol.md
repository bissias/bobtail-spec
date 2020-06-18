# Working Specification for Bobtail

1. Miners assemble a sub-block header and grind on its nonce in the same manner as in Storm; we call the resulting hash values *proofs* as in Bobtail. The contents of the sub-block are identical to a Storm delta block with the following exceptions.
   - The coinbase is now called a *proofbase* and has the following properties.
     - The output is null, i.e. no coin is allocated to the miner at this stage.
     - The first input, `vin[0]`, is also null.
     - Subsequent inputs, `vin[i]` with `i > 0`, are used to store *parent hashes*, which we describe below.
   - The Merkle tree contains only the proofbase (as the first transaction) and *new* transactions, not any transactions from its parents.
2. Parent hashes for each sub-block are, similar to Storm, earlier proofs having the same parent block.
3. The weak PoW target `w` for sub-blocks is set to a value such that the kOS will fall below `w` with probability 0.99999.
4. When a proof is generated that meets `w`, the miner disseminates it (along with associated header and transactions) to other miners.
5. Miners maintain a list of the lowest `k` proofs seen thus far, along with all the parent hashes for those proofs.
6. Proofs and their parent hashes comprise one or more DAGs, where an edge from A to B in the DAG indicates that proof B uses proof A as support.
7. A *valid subgraph* is any weakly connected subgraph of the DAG where the union of transactions specified by the constituent proofs contains no conflicting transactions.
8. The mining process continues until it is possible to assemble a block, which is carried out by a miner when:
   - He can assemble exactly `k` proofs (not necessarily his own) comprising a valid subgraph and having mean value less than target `t`.
   - He was the miner of the lowest proof value in the valid subgraph (the 1OS).
9. Once a miner is capable of assembling a block, he:
   - Adds all proofs to the block
   - Creates a new Merkle tree from the union of transactions stipulated by all `k` proofs in the block and adds the root of the tree to the block
   - Creates a coinbase transaction that divides rewards evenly among all k proofs
   - Signs the block and propagates it to other miners.
10. In the event of a block race, the winning block is the one with the highest score which is calculated as:
    - The score of a proof that has no descendents is 0.
    - For a proof with descendents, each descendent of generation `i` has score `i`, e.g. children score 1 point and grandchildren score 2 points.
    - The score for a block is the cumulative score of all `k` proofs in the block.

## FAQ

### Q. Why doesn't a miner on the Bobtail chain stop after mining one proof and switch to BTC?

A miner with fraction `x` of the hash rate expects to mine `xk` proofs every block (each having the same expected reward). In order for this share of the reward to be achieved on average, the miner must always mine for the entire time period until a block can be assembled. Due to variance in the mining process, the miner will occasionally mine more or less than `xk` proofs. But the average will remain `xk`. Suppose, on the occasion that the miner is lucky and mines `xk` proofs with some expected time `t` remaining, the miner switches over to BTC. This action will slightly reduce the expected reward for the miner conditioned on him already having mined `xk` blocks. Accordingly, increase in profit on BTC during time `t` will manifest as an opportunity cost on the Bobtail chain.
