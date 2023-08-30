# TVM and memory layout

###### tags: `Chapter 2`

*In this lesson we will talk about TON's memory layout and TVM operation.*

:checkered_flag: Let's start with the unique feature of TON that is called ***cell***.

## Cells.

> ###### What is a cell?
> :large_orange_diamond: ***Cell - small building block of the entire data structures in TON blockchain.***
> Each ***cell*** has up to 2023 bits of data :small_orange_diamond: and up to four references :link: to other ***cells***. And this allows you to use ***cells*** to build arbitrarily complex and nested data structures. 

> :heavy_exclamation_mark: Thus, it is not possible to allocate an array of arbitrary size in TON, as can be done in Ethereum. In TON you have to work with a ***tree of cells*** :deciduous_tree:.

Storage model in TON is obviously giving some challenges to the developers because now you have to split your data in those chunks of 2023 bits which is slightly less than 128 bytes. 
:question: And you have to think *how flat or how deep* you want to build the tree of your data. 

## Why cells?

> :exclamation: ***The cool result of this design decision is that the entire state of the blockchain can be effectively Merkle-ized which means you can create the Merkle proof, the cryptographic proof of any portion of the data in the blockchain at any state of it.***

<u>This is crucial when you scale out to a large system :globe_with_meridians: with multiple shards and separated validator groups and you need to verify that some groups behaved correctly and didn't break the rules of the system. And this is where the effective compact Merkle proofs are necessary to prove any misbehavior by any participant in the system.</u>

## More types in TVM.

##### TVM has enough types to work with:
- <u>Cells
- Integers
- 257-bit integer</u> (allows you to represent a wide range of integers suitable for cryptographic work and for financial operations :sunglasses:)

All the data types that TVM works with could be *read by the code in the contract out of its own storage, manipulated on the stack and then the new storage could be created with the out protection*. 
If the execution was successful :heavy_check_mark: then the TVM is unloaded from memory and the new state of a contract is stored in its place. 

## Memory layout.

Let's talk about the memory layout available for the contract.

<u>*The only available option for your memory layout is a tree of cells in a contract.*</u>

:question: But how to store lists, dictionaries and sets in the system?

:worried: *It's not very trivial to do because the cells are very compact and they form a tree.*

> To help developers with this, TON comes at the level of TVM and at the level of FunC, the higher level language, with tools that help you work with hashmaps in a more effective manner and use cells as the underlying implementation.

## Key to scalability in TON.

> :exclamation: ***Scalability in TON is to limit the amount of work that is done in any single place because TON is scalable across contracts, but each individual contract is not scalable out of the box by itself.***

:question: What are the key things to keep in mind?

> 1. ***It is totally acceptable to have nested cells for small amount of data.***
> *If you make a system with a very few participants, let's say a multi-signature contract, then you can store this data in your hashmap right into the contract and it will be reasonably small. However, if you're building a system with millions of users then you should think about using tokens to represent users participation and avoid storing the lists of these users in your contract altogether.*

> 2. ***Cells have a built-in deterministic hashing scheme that allows you to identify uniquely any cell in any part of the tree and this is used both for data deduplication and compression.***
> *For instance, when the contract runs out of money to pay for the rent, its current state is offloaded from the blockchain. It's completely forgotten but the network still stores the hash of the storage cell. This means that the user could come later and provide the original :evergreen_tree: tree of cells that matches this hash and re-instantiate the contract and this way the state of the contract is fully preserved by the hash of its entire storage.*

## Conclusion.

##### Here are some of the major results:

- <u>Everything is built out of :small_orange_diamond: cells.
- On top of the cells you have types provided by :computer: TVM.
- TVM is instantiated for each contract for each message that is processed by the :pencil: contract.
- You should be aware of the :diamond_shape_with_a_dot_inside: costs associated with large data structures and deep chains of cells that may incur higher than usual costs on your contract execution.</u>
