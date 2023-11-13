# History Network

***Note that this network is not yet fully production-ready***

The chain history data consists of historical **block headers**, **block bodies** (transactions and ommer) and **block receipts**.
It is expected that this network will one day allow clients that have pruned historical blocks to retrieve them on-demand from this network, and nodes may be able to use blocks downloaded from the History Network to sync, rather than maintaining a local copy of the blockchain. 

The History Network is a [Kademlia](../kademlia.mdx) DHT that uses the [Portal Wire Protocol](./portal-wire-protocol.md) to establish an overlay network on top of the [Discovery v5](../discovery.mdx) protocol.

The History Network also provides a [hash accumulator](../../hash-accumulators.mdx) that enables clients to verify that a header is at a particular block number in the pre-merge Ethereum mainnet chain.

## Data

Portal clients can retrieve the following data types from the History Network:

* Block headers
* Block bodies
    * Transactions
    * Ommers
* Receipts
* Header epoch accumulators (pre-merge only)

### Data Types

#### Block Header

Block header data can include: 
- A `BlockHeaderWithProof`
- A `BlockHeaderProof`
- An `AccumulatorProof`

The `BlockHeaderWithProof` is the actual block header data wrapped in a higher level structure with a Merkle proof demonstrating its inclusion in the canonical Ethereum chain. The proof is of type `BlockHeaderProof`, which is a `Union` that can be either `None` or `AccumulatorProof`. The `header` field in `BlockHeaderWithProof` contains the RLP encoded header encoded as an [SSZ](../ssz.mdx) bytelist.

For pre-merge headers, clients SHOULD NOT accept headers without a proof as there is the `AccumulatorProof` solution available.
For post-merge headers, clients SHOULD accept headers without a proof, and determine validity using the Beacon network.

Block headers are retrieved from the network by specifying a content key generated by concatenating the identifier `0x00` and the target block hash.

#### Block Body

The block body contains the complete list of transactions that were included in the block and the list of ommers (blocks produced simultaneously but not included in the canonical chain). The encoding for block bodies changed at the Shanghai upgrade (see [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895)), meaning Portal clients now have to detect whether a block is pre-Shanghai or post-Shanghai to determine how they should be decoded. Clients that request block bodies from the History Network execute this sequence of operations:

- Receive raw block body content.
- Fetch the block's header from the network.
- Compare header timestamp against `SHANGHAI_TIMESTAMP`.
- Decode the block body using either pre-shanghai or post-shanghai encoding.
- Validate the decoded block body against the roots in the header.

Block bodies are retrieved from the network by specifying a content key generated by concatenating the identifier `0x01` and the target block hash.

#### Receipts

Receipts record the outcome of transactions, including transaction logs. 
Portal provides RLP-encoded receipts as SSZ bytelists.

Receipts are retrieved from the network by specifying a content key generated by concatenating the identifier `0x02` and the target block hash.


See section 4.3.1 in the [yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf)


#### Epoch Accumulator

Portal clients can request the accumulator hash for any epoch. This is the root hash for a Merkle trie with blocks as leaves and is unique to a specific epoch. Proofs are generated by generating a path from an individual block hash to the epoch root hash. The epoch hashes are added as leaves in a master accumulator.

Epoch accumulator hashes are retrieved from the network by specifying a content key generated by concatenating the identifier `0x03` and the target epoch hash.

Read more on [Hash accumulators](../../hash-accumulators.mdx)

## Data Retrieval

The network supports the following mechanisms for data retrieval:

* Block header by block header hash
* Block body by block header hash
* Block receipts by block header hash
* Header epoch accumulator by epoch accumulator hash

> This sub-protocol does **not** support retrieval of transactions by hash, only the full set of transactions for a given block. See the "Canonical Transaction Index" sub-protocol of the Portal Network for more information on how the portal network implements lookup of transactions by their individual hashes.


## Specification

You can find the History Network specification on the [Portal Network Github](https://github.com/ethereum/portal-network-specs/blob/master/history-network.md).