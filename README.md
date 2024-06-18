## Flattening the Merkle Patricia Trie Data Structure ##

### Abstract ###

The Merkle Patricia Trie (MPT) data structure is an essential part of the Ethereum network and is used to store network state such as accounts and storage as key-value pairs. Although the data structure lends itself well to efficiently producing cryptographic proofs of data for quick verification, looking up a key is slow and usually takes several database lookups since in a non-flat, tree-like MPT,  data is stored in layers of nodes that need to be looked up one-by-one in order to fully query a single key-value pair for meaningful data. Flattening the MPT data structure by keeping data as simple key-value pairs would be much more efficient, but verifying the data would become computationally inefficient since changes to data would invalidate all calculated intermediate node hashes and we'd need to recalculate them from scratch. This change proposes a different method of organizing key-value data in a flat MPT structure by key prefix, ensuring logarithmic complexity for modifications, insertions, deletions, and verification, as well as constant time key lookups, although with the added overhead of additional memory usage for keeping two sets of data, including the leaf data that contains meaningful data about network state, and also the calculated set of nodes used for proof creation and verification of data.

### Introduction ###

An [MPT](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/) stores key-value data by breaking down keys into smaller chunks, such as nibbles, which are used to construct nodes down to the leaf node where the value is stored. Data lookup begins at the root node, querying by hash to reveal child nodes, which are organized by the subsequent nibble in the key. This process continues through intermediate nodes until the leaf node with the desired state data is reached. On the Ethereum mainnet, this method typically requires 7-8 internal node lookups and multiple disk accesses to retrieve a single key [1]. A traditional MPT offers O(logn) complexity for lookups, modifications, insertions, deletions, and verification.

Storing data in a flat key-value mapping can drastically reduce lookup time from O(log n) to O(1) by minimizing the number of disk accesses. However, this approach makes data verification computationally inefficient, as an insertion or deletion operation would invalidate all subsequent hashes, necessitating a full rehash of the dataset, since there is no hierarchical path to leverage. A change in any single key-value pair does not have a localized impact but rather affects the entire dataset.

### Goals ###

The primary goal is to enhance the speed and efficiency of network state lookups, thereby enabling faster and more performant network clients. This is achieved by developing a flat MPT design that allows O(logn) complexity for modifications, insertions, deletions, and verification, while providing O(1) lookups for keys.

### Approach ###

To minimize disk accesses for key lookups, we propose maintaining a flat key-value mapping, reducing lookup complexity to O(1). However, recalculating hashes for the entire dataset upon each insertion or deletion remains a challenge, as it invalidates already-calculated, existing hashes.

By organizing leaf data according to key prefixes, we can mitigate this issue. Insertions and deletions will affect only the path from the affected leaf node to the root, rather than shifting all leaf nodes. This confines the necessary recalculations to the modified path, preserving the hashes of unaffected nodes.

This approach involves maintaining two datasets:
* **Flat key-value mappings of leaf state data**: Facilitates O(1) key lookups.
* **Calculated intermediate nodes of the MPT**: Allows O(log n) complexity for modifications, insertions, deletions, and verification from leaves to the root.

## Tradeoffs

Maintaining a flat key-value mapping alongside a non-flat MPT entails increased memory usage for the additional flat leaf data. This tradeoff results in:
* Lookup time reduced from O(logn) to O(1).
* Write complexity increased to O(1 + logn) from O(logn).
* Disk storage requirements increased from O(nlogn) to O(n + nlogn).

## Sources

[1] Ask about geth: Snapshot acceleration. Ethereum Foundation Blog. (n.d.). https://blog.ethereum.org/2020/07/17/ask-about-geth-snapshot-acceleration 

[2] “Patricia Merkle Trees.” Ethereum.org, ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie.
