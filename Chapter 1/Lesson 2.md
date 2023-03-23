# Chapter 1. Lesson 2. Blockchains, accounts, transactions in TON

###### tags: `Chapter 1`


## Blockchain accounts and transactions

In the previous lesson, we did an overview of the entire ecosystem. Now we're going to dive specifically into the architecture of the blockchain itself, to give an overview of a blockchain itself. 

### How blockchains work

So what is the block and what is the chain in the context of blockchain? TON blockchain is a ledger of state transitions.
It's kind of a more general idea than just a ledger of transfers. It's a ledger of changes in the state of arbitrary accounts. Every **smart contract** is also called an **account**. 

An account has its own place, it has its **own storage** and it has its own **unique identification address**, so we call it an address or a contract ID.

Another way to describe a contract is an **account** that holds **arbitrary data**, **TON coins** and the **program code**. 

It's up to the developer to design the logic of the contract by using any code they want within the syntax supported by the virtual machine called TVM. 

Developers are provided with pretty much unlimited flexibility through the lifetitem of a contract. They can change the data, they can even change the code of the contract in certain scenarious and of course - they can transfer the coins around. It's important to note that these contracts are self contained, they have no visibility into anything outside themselves. This is what ensures the infinite scalability of the TON blockchain as every contract is self-contained - it can only receive the incoming message from some other contracts or from the outside world. The contract will then process this message by program code and change it's own state according to the results of this incoming message processing.

As TON blockchain is asynchronous, it cannot go and read the state of any other contract and immediately get the result. In order to use some other contract's state, the contract has to  send a message and maybe later, it will receive the desired reply and it's program code has to be ready to process this reply along with some other message from other contract. This concept makes those contracts look very similar to computers on the internet. So every computer is independent, they have their own state, their own programmed logic and they have no idea when and how their messages will be processed.

### The Guarantees and Limitations of Blockchain Technology

There are some guarantees that the blockchain provides you. You could verify the address of any incoming message and be sure that behind it was specific code that you might trust. Similarly, if you send a message to a specific address, you have a cryptographic assurance that there will be very specific code that you may want to trust. Other than that, the network doesn't really guarantee any certain delays for the reply, due to various reasons. It might be very well that the participants in the desired reply are scaled out to billions of computers. Messages that participate in forming the reply might be routing over multiple blocks and takes time for the final reply to be formed. It might be also a case that other contracts ran out of money or somethin unexpected can happen. So while developing your contract you need to only focus on your own contract's logic and state.

### Messages and Transactions

So what are messages and what are transactions. The thing that happens between the contracts is called a message, it carries a little bit of coins and arbitrary data to a specific destination, the contract ID or address.
When the message arrives to the contract it's processed, the contract updated state, and emits new messages, this whole bunch of activity on a contract is recorded as a transaction. Think of this, if you have a chain of messages from contract A to contract B to contract C, then you have a message in between A and B. But you have transactions on A and transaction on B. And then if you have another message from B to C, that's the second message and the third transaction that is recorded on contract C. So if you have this chain with two messages, then you have three transactions and two messages. You also have a third message that is an external message that activates this chain.

### The Process of Sending and Recording Transactions on the Blockchain

Because the blockchain by itself is just a data structure it can, like live its own life, doesn't have its own timer. It has to be signaled from outside. So the user has to press a button on the wallet, create some data object that encapsulates a message and some signature, maybe some other instructions, and sends it to the network validators to receive this message and record it as an incoming message to the first contract to which it was addressed. Typically, it's the user's wallet contract, then this is recorded in the first transaction. So you have this diagram where you have an external message, one, then construct a processor to emit an internal message to and this fact is recorded as a first transaction and then the contract B receives this internal message and changes its state to emit another message. This update record is recorded as a transaction number two. So the user has to press a button on the wallet, create some data object that encapsulates a message and some signature, maybe some other instructions, and sends it to the network validators to receive this message and record it as an incoming message to the first contract to which it was addressed. Typically, it's the user's wallet contract. And then this is recorded in the first transaction. So you have this diagram where you have external message, one, then contract A processor to emit internal message two and this fact is recorded as a first transaction. And then contract B receives this internal message and changes its state to emit another message. And this update record is recorded as a transaction number two. So you can see that every contract has its own transactions, which means that every contract is effectively sort of like a blockchain that has its own chain of state updates and its own transactions. And this is quite helpful, because now you can imagine that you could process and verify the details, chains of transactions rather independently. The only question remains, how do you route those messages and kind of synchronize that the message is actually delivered and how do you prevent double spends?

### Double Spends

So let's start with double spends first. So in the TON blockchain, you need a consensus, you need to make sure that if you have two equally valid ways to make a transfer, like I can send money to, to person A, or I can send the money to person B, either of those are completely valid transactions. And from the perspective of the entire network it doesn't matter which one is recorded, the only thing that's important is that only one of them is recorded. And for this, we need consensus. So in TON, to make consensus scalable, they use proof of stake, where you have a set of validators that put the large amount of TON coins as a security deposit as a security bond, that guarantees that they don't misbehave. If any validator misbehaves or is switched off, they're punished with small or big fines that are taken out from this stick.

### TON's Scalable Validator Management and Sharding Mechanisms

There are various mechanisms to prove to all the other validators whether this one was offline or misbehaving. Also in the setting, the set of validators can be shortened. If the load on the system increases, and this number of accounts grows, then the sets of validators could kind of split in subgroups that could intersect and still say the even account numbers will be handled by one group and odd account numbers will be handled but by another and TON allows this sharding to be pretty much unlimited down to very few accounts depending on the load and the message routing is handled with various mechanisms.


### Two-Tier Blockchain System in TON for Transaction Processing

And finally when all these like subsets agree on their parts of these transaction histories, then these agreements are recorded in one non scalable blockchain called Master chain that every validator is participating in. So in TON, you have this kind of two tier system, you have the master chain and you have the base chain. The base chain is the one that's kind of virtual because it actually contains all of these various accounts that could be sharded or can be immersed and split at any moment of time. In the master chain is this one non shareable chain that is expensive and not scalable, but also it's a very low load thing, it's only used to do the transactions on some configuration of the network. And the snapshots of all these sub chains from these validator groups. So that's kind of like the picture of the blockchain. And this kind of explains the latency in the transaction processing. So when you process a transaction on the blockchain, this takes around five to six seconds to be committed there, and then it takes another five to six seconds, for the same consensus kind of synchronization reason to commit this update in the master chain.

### Delays and Commitment Process in Global Transactions

So the total delay to make a globally visible commit of this transaction would take around 10 to 12 seconds. And this is just for the initial transaction, then if the transaction creates a long chain of messages, those could be routed, you know, over time, and eventually be committed in maybe the same or maybe different blocks. So if you're only interested in the final transaction, then that could take even longer. However, in the usual cases, the chains are quite short. And also, you often don't have to worry about committing the last one, because once you've processed the first transaction on the chain, then for many applications, it means that the rest of the chain will be completely determined and will not change any results.

### Conclusion

So you can look at your account status, see that the center created there they're part of the transaction, then you can predict that sooner or later, you will receive your asset or something specific will happen. The only difference is that what will be the delay otherwise, for many applications is just fully deterministic. You don't have to wait all the time to confirm the payment. You could just confirm the first part of this transaction graph. So that's the anatomy of the blockchain in term.





