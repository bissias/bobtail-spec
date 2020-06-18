# Background

## Storm

Storm implements *delta blocks* which have many of the same properties as regular blocks with the following exceptions.

1. To be valid, a delta block must have PoW that meets a certain *weak threshold*.
2. A delta block is considered *strong* when it meets the usual block target.
3. Delta blocks have zero or more *weak ancestors*, which are earlier delta blocks having the same strong parent.
4. When a delta block is propagated only new transaction, those not included in weak ancestors, need be propagated with it.

Storm also changes the Bitcoin Cash protocol so that a strong delta block having the most weak ancestors will beat every other strong delta block having the same strong parent in the event of a block race.

The Storm protocol has two key advantages over the conventional Bitcoin protocol. First, the block orphan rate is reduced because, generally, fewer transactions need to be sent in order to convey a strong delta block (see 4 above). And second, the network enjoys a form of pre-consensus in that network participants can infer that a transaction will eventually be mined in a strong block if it is included in the majority of delta blocks.

Despite its benefits, there exist some limitations to the Storm protocol. First, because weak ancestor count is the critical determinant in resolving block races, it is not clear that the protocol provides sufficient motivation for miners to share their delta blocks with other miners. Second, the pre-consensus provided by Storm is purely speculative in that there is no reason that the final strong block needs to include any weak ancestors or the transactions the specify.

## Previous Bobtail

Bobtail is a more fundamental change to the Bitcoin Cash protocol, which alters the PoW mining criterion. Specifically, a block is mined only when the mean of the lowest `k` block hashes (having the same parent) fall below the target `t`. The general properties of Bobtail are listed below.

1. Miners assemble a block and grind on its nonce in the same manner as in Bitcoin; we call the resulting hash values *proofs*.
2. Headers additionally include a *supporting proof*, which is the lowest proof value (with the same parent block) yet seen network-wide.
3. When a sufficiently low proof is generated, the miner disseminates it (along with associated header and transactions) to other miners.
4. Miners maintain a queue of the lowest `k` proof values seen network-wide.
5. The mining process continues until it is possible to assemble `k` proofs whose average falls below `t` (proofs should be included in FIFO order).
6. A miner can assemble a valid block, child to parent block `P`, provided that:
   - He can assemble `k` proofs (not necessarily his own) having parent `P` and mean value less than `t`.
   - He was the miner of the lowest proof value (the 1OS).
   - The supporting proof for each proof (excluding the 1OS) has parent `P` and value at least as large as the 1OS.
7. Once a miner is capable of assembling the block, he signs it and propagates it to other miners.
8. Only the transactions in the Merkle tree of the 1OS are considered part of the block.
9. Each miner who generated a proof in the block is awarded one unit of primary reward.
10. Any miner whose supporting proof points to the 1OS also receives a bonus reward.

### DS avoidance rule

By construction, Bobtail punishes miners who produce proofs that include doublespending transactions in their Merkle tree. This provides incentive for miners to avoid mining transactions that conflict with other transactions. A simple rule can help to that end. When a miner first sees a transaction, he waits 5 to 10 seconds before mining it. During that time period, he monitors the network for a conflicting transaction. If none arrives, then he begins to mine the transaction. And if a conflicting transaction does arrive, he mines neither. Finally, if a miner is already mining a given transaction, and a conflicting transaction arrives, the miner discard the conflicting transaction. These simple rules ensure that protocol complaint miners are highly likely to settle on the same set of valid transactions.

## Combining Storm and Bobtail

The Storm protocol provides some pre-consensus, but lacks concrete doublespend protection. Meanwhile, the Bobtail protocol reduces variance in inter-block time, and as a result, reduces the blockchain's susceptibility to doublespend and selfish mining attacks. However, as specified, it does not provide a pre-consensus mechanism. Worse, by only including the transactions specified by the 1OS, Bobtail destroys zero-confirmation acceptance for transactions that arrive after the 1OS (half of the transactions in the block on average). Therefore, we seek a combination of the two technologies that preserves the benefits of each without suffering from their drawbacks. We propose the following approach to combining the two protocols.
