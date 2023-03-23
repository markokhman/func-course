# Chapter 1. Lesson 2. Blockchains, accounts, transactions in TON

###### tags: `Chapter 1`


## Blockchain accounts and transactions

In the previous lesson, we did an overview of the entire TON ecosystem. Now we're going to dive specifically into the architecture of the blockchain itself.

### How blockchains work

So what is the block and what is the chain in the context of blockchain? TON blockchain is a ledger of state transitions.
It's kind of a more general idea than just a ledger of transfers. It's a ledger of changes in the state of arbitrary accounts. Every **smart contract** is also called an **account**. 

An account has its **own storage** and it has its own **unique address**. **Accounts** also stores **TON coins** balance and the **program code**. 

It's up to the developer to design the logic of the contract by using any code they want within the syntax supported by the virtual machine called TVM. 

Developers are provided with pretty much unlimited flexibility through the lifetime of a contract. They can change the data, they can even change the code of the contract in certain scenarios and of course - they can transfer the coins around. It's important to note that these contracts are self-contained, they have no visibility into anything outside themselves. This is what ensures the infinite scalability of the TON blockchain as every contract is isolated from the others - it can only receive the incoming message from some other contracts or from the outside world. The contract will then process this message by program code and change it's own state according to the results of this incoming message.

As TON blockchain is asynchronous, it cannot go and read the state of any other contract and immediately get the result. In order to use some other contract's state, the contract has to  send a message and maybe later, it will receive the desired reply. This concept makes those contracts look very similar to computers on the internet. So every computer is independent, they have their own data, their own programmed logic and they have no idea when and how their messages will be processed by others.

### The Guarantees and Limitations of Blockchain Technology

There are some guarantees that the blockchain provides you as a developer. You could verify the address of any incoming message and be sure that behind this address there is specific code that you may trust. Similarly, if you send a message to a  address, you have a cryptographic assurance that there will be  specific code behind this address. The network guarantees that the message will be delivered but doesnt gauarantee how long this will take. The chain of messages could spend thousands of shards and take multiple blocks to finalize the transaction.

It may also be a case that some intermediate contracts run out of money and fail processing their messages, so when you develop the contract you need to focus on your own contract's consistency and the way it interacts with other contracts.

### Messages and Transactions

So what are messages and what are transactions. The thing that happens between the contracts is called a message, it carries a little bit of coins and arbitrary data to a contract address.
When the message arrives to the contract it's processed, the contract updated state, and emits new messages. All this activitity on a contract is recorded as a transaction. Think of this, if you have a chain of messages from contract A to contract B to contract C, then you have a message in between A and B. But you have transactions on A and transaction on B. And then if you have another message from B to C, that's the second message and the third transaction that is recorded on contract C. So if you have this chain with two messages, then you have three transactions and two messages. 

### The Process of Sending and Recording Transactions on the Blockchain

The blockchain itself is just a data structure. To change it, there must be a signal from the outside. The user presses a button in the wallet, the wallet creates an external message to a destination contract. The wallet sends this message to the validators who apply it to the contract.

Most often, users send external messages to their own _wallet contracts_, which is their first transaction. Such messages contain embedded messages to other contracts. So the second transaction in the chain is a message from the wallet contract delivered and processed by the actual destination contract.

So you can see that every contract has its own transactions, which means that every contract has its own blockchain. And this is quite helpful, because the network can process and verify transactions independently from each other. The only question remains, how do you route these messages, and how do you prevent double spends?

### Double Spends

So let's start with double spends first. I can send money to, to person A, or I can send the money to person B, either of those are completely valid transactions. And from the perspective of the entire network it doesn't matter which one is recorded, as long as it is only one. And for this, we need consensus. To make consensus scale horizontally, TON uses proof of stake, where you have a set of validators that put the large amount of TON coins as a security deposit as a security bond, that guarantees that they don't misbehave. If any validator misbehaves or is switched off, they're punished with small or big fines that are taken out from this stick.

### TON's Scalable Validator Management and Sharding Mechanisms

There are various mechanisms to prove to all the other validators whether this one was offline or misbehaving. Also in the setting, the set of validators can be shortened. If the load on the system increases, and a number of accounts grows, then the sets of validators could split in subgroups. For instance, even account numbers will be handled by one group and odd account numbers will be handled but by another. TON allows this sharding to be pretty much unlimited down to individual contracts depending on the load.


### Two-Tier Blockchain System in TON for Transaction Processing

When all the validator groups achieve agreement on their parts of the blockchains, they record the state of the accounts on a central non-shardable blockchain called Masterchain where every validator participates. So in TON, you have this kind of two tier system, you have the master chain and you have the base chain. The base chain is the one that's kind of virtual because it actually contains all of these various accounts that could be sharded or can be immersed and split at any moment of time. In the master chain is this one non shardable chain that is expensive and not scalable, but also it's a very low load thing, it's only used to do the transactions on some configuration of the network. And the snapshots of all these sub chains from these validator groups. So that's kind of like the picture of the blockchain. And this kind of explains the latency in the transaction processing. So when you process a transaction on the blockchain, this takes around five to six seconds to be committed there, and then it takes another five to six seconds, for the same consensus kind of synchronization reason to commit this update in the master chain.

### Delays and Commitment Process in Global Transactions

So the total delay to make a globally visible commit of this transaction would take around 10 to 12 seconds. And this is just for the initial transaction, then if the transaction creates a long chain of messages, those could be routed, you know, over time, and eventually be committed in maybe the same or maybe different blocks. So if you're only interested in the final transaction, then that could take even longer. However, in typical scenarios, the chains of messages are quite short. Also, you often don't have to worry about committing the last one, because once the first transaction is recorded, you may often predict the results of all the subsequent transactions. For instance, when accepting simple payments with tokens, you may detect the confirmation of the initial transaction performed by the user and not worry about the funds arriving at your destination wallet, which may take four transactions.

### Conclusion

To conclude, in TON each contract is its own account with data and code. Contracts are isolated from each other and communicate by sending messages. This allows blockchain to be infinitely sharded. Consensus of validators guarantees that messages are eventually delivered, not replayed and funds are not double-spent. And finally, most operations take multiple transactions over a number of contracts, but it takes only seconds to confirm the very first transaction.





