# Chapter 1. Lesson 1. Blockchains, accounts and transactions

### What is blockchain 

TON Blockchain is designed as an _infinitely scalable_ platform for _decentralized applications_.

This sounds pretty abstract, so let’s unpack these two requirements. 
We will start with the most basic idea and build up to reach the level of complexity of TON.

### Bitcoin model: first generation blockchain

The first and the most successful blockchain, Bitcoin, has the simplest design. Bitcoin solves one problem: how to enable people to hold accounts and exchange digital cash (or digital gold, if you prefer) in a fully decentralized manner? 

To do that, Bitcoin offers these components:

1. Public key-based cryptographic identities: each "account" is simply a public key that is used to authorize any transfer of coins from it.
2. Transactions: are data structures that contain cryptographic signatures authorizing transfer of coins from one public key to another. Transactions form a cryptographic chain of coins, and since amounts can be split and merged, that chain is actually a directed acyclic graph.
3. Public distribution: all nodes verify all transactions and ignore unauthorized or malformed transfers.
4. A consensus protocol: to prevent double-spending, all nodes in the network also verify proof-of-work stamp that helps select the "heaviest" version of the transaction graph at any time, thus allowing to (eventually) work around synchronization issues, attacks on the network etc.

Bitcoin went a little bit further and added ability to customize authorization logic: instead of plain public keys, each "account" is actually a small predicate. Most often that predicate is just "check signature for public key X", but it may also contain multi-key checks (aka "multisignature contract"), timelocks and a few other conditions.

### Ethereum model: second generation blockchain

5 years after Bitcoin, Ethereum came on stage and offered a further generalization on top of this model. The goal of Ethereum is to allow unrestricted innovation not only in the ways to work with the money (coins), but also allow people to define their own financial assets. Ethereum adds the following ideas to Bitcoin:

1. Each account’s state is not just the predicate + amount of coins, but an arbitrary data store + arbitrary Turing-complete code + amount of coins.
2. To make execution safe, each operation has a precise definition of a cost in "gas units" that are paid by the account.
3. Each account’s code may call into any other account, thus turning the blockchain into a giant library of programs. 
4. Each program’s duty is to protect its storage and balance and correctly respond to external invocations. 

Ethereum opened the gates to "decentralized finance" with such powerful concept.

However, both Bitcoin and Ethereum did not solve the scalability problem. Consensus protocols of Bitcoin and Ethereum require every node to verify all transactions. As usage grows and smart contract applications become more interconnected, the pressure on the entire network grows super-linearly and fees for transaction execution skyrocket.

### TON model: third generation blockchain

TON is a third-generation blockchain that adds a few more ideas and a couple of tradeoffs to all of the above:

1. Unlike Ethereum, calling into "other accounts" is done by issuing asynchronous "messages". That is, one program cannot wait for another to yield a result. In fact, a program cannot see any state of the blockchain apart from its own. It can only emit and receive messages.
2. This allows thinking of each account as running its own blockchain. 
3. Consensus protocol is designed to be two-tier: at the account level (to prevent double-spending of the individual accounts) and at the global level (to route messages between accounts).
4. To enable horizontal scaling, TON uses Proof-of-Stake scheme with various intersecting subgroups of validators that take on subsets of accounts to validate. This allows spreading the cost of verification instead of multiplying it.
5. All possible costs are precisely accounted for: not only you pay fees for computation, but also fee for routing of messages and the rent for data storage.

### Summary

In TON, each _account_ (aka _contract_) is a "program + data + coins", like in Ethereum. TON tries to keep as little logic at the built-in level and delegate as much as possible to the "contracts" level: that is, permit users to customize and extend the platform to their liking.

Contracts have their own history and communicate with each other and external world via _messages_. 

_Transactions_ record the _incoming_ message and the results of its processing: the _new state_ of the contract + _outgoing_ messages that it emitted.

This scheme allows verifying contracts completely independently from each other. Meaning, the blockchain is already _sharded_ at the level of each account. 

Consensus protocol is proof-of-stake that forms two blockchains: _masterchain_ and _workchain_ (also known as _base chain_). Masterchain never shards, while workchain can dynamically split and merge back _shardchains_: groups of contracts with the common set of validators.

Masterchain is used for infrequent configuration transaction and for recording routing of the messages within the workchain. This means masterchain, like Bitcoin, does not scale, but is also not doing much. While workchain, like Ethereum, contains all the complexity of custom smart contracts, but scales horizontally.

_Validators_ are nodes that lock up their _stake_ (security deposit) under promise to correctly verify transactions and sign off blocks. The "valid blockchain" is the one with two thirds of signatures.

