
# Concept of Contracts and Accounts

###### tags: `Chapter 1`

*Let's do a deep dive into accounts and contracts.*

## Contracts

> ##### In the previous lesson, we learned the following things about contracts:
> - *Account and contract at the low level in TON are interchangeable terms.*
> - *Contract is the data, code, and Toncoin balance.*
> - *Contracts receive incoming messages, process them and emit outgoing messages and also change their own internal state.*

But let's dive a little bit deeper. :whale:

> :mailbox_with_mail: **Contract also has the identifier or address.** 
> *This address is a cryptographic hash of the contract's initial data and initial code.*
> :question: Why is that? 
> :ballot_box_with_check: You don't want to change the address whenever the state of the contract changes, and that's why the address is uniquely identifying the very initial state with which the contract was created. 

> :tent: **The second important aspect of the contracts is their locality.**

All of this data is <u>fully encapsulated and from the perspective of the program code</u>, it only sees the storage of the contract itself and its balance. It cannot see the state of all the other contracts. This means that whatever changes are happening to a contract in one transaction are completely independent from changes to another contract in another transaction somewhere else on the blockchain.
 ***This is the key to infinite scalability of TON blockchain.***

## What can be done with contracts?
:question: What can you build with contracts? 

1. First of all, the <u>contracts allow you to build the user accounts</u>. 
*In TON, every user's account is actually a custom wallet contract.*

2. Secondly, multi-signature contract that operated by multiple user wallets.

3. Third, the contracts are used to build some things that you don't typically think of as a contract. 
*For instance, tokens*.

## Tokens in TON

*In TON, the token that you can transfer is really a separate contract that has its own state that specifies some attributes of this token.*

> :moneybag: One of these attributes is the owner.

*Whenever you want to change the ownership of a token, you have to send the message to this token that specifies the new owner. Then the token will check that the message is sent by the appropriate owner, change its owner to a new one, and be done with it.*


## Protecting TON

*To protect the network against denial-of-service attacks, all the contracts need to pay for their operation.*

This payment (it is also called fees) consists of many, many parameters that cover the rent, the execution costs, the message routing, and some other things. 
*Let's dive in the most important ones.* :whale:

> :exclamation: Every time you execute the code on the contract, you invoke ***gas costs*** – every instruction in the virtual machine, the TVM, has a designated cost in the abstract units called ***gas***.
> :gem: At the network level, there is a parameter that is called ***gas price*** that specifies how many Tonсoins you have to pay for each instruction.
> :hourglass_flowing_sand: *The longer your program runs, the more gas costs it will incur, and this cost is deducted from the contract's balance.*
> :x: *Whenever the balance runs down to zero, the execution aborts, and the transaction fails.*
> :heavy_dollar_sign: Gas cost makes sure that you cannot impose unbounded execution costs on the whole network without paying for it.

**But there is another important cost that is called rent!**
> :exclamation: TON makes sure that for every bit of data, and for every second of the life of the contract, there is a designated payment called **rent** that is deducted from the contract balance.


## Considerations when designing smart contracts.

There are *two most important considerations when designing smart contracts*:
1. *Gas costs* of the execution. 
2. *Rent* that the contract has to bear over its lifetime.

## Frozen contracts.

> If the contract runs out of funds and because of the rent, then it may become ***frozen*** :cold_sweat:.

This means that the network will offload all of its data and replace it with a cryptographic hash of its latest state. In this case data is not lost, but the network optimizes the storage :cd: and offloads expensive data out of the storage of the validators. When the user of this contract comes along and wants to reinstate :crystal_ball: the contract to unfreeze it, they have to provide their own snapshot of the data that matches this hash.
