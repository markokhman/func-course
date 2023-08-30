# Scalable contracts

###### tags: `Chapter 2`

*Let's talk about scalability.*

> :gem: **In TON, you really have a very specific guarantee around scalability. TON guarantees you that it can scale down to granularity of individual contracts.**

<u>Therefore, operations on one contract will not create a bottleneck :baby_bottle: for operations on another contract. And then the TON network obviously will take care of all the message routing :bullettrain_front: between them.</u>

> :exclamation: **It's your job to take advantage of the entire scalability of blockchain and not create bottlenecks in your own contracts!**

## Problems with monolithic contracts.

*One of the obvious problems for people who are coming from traditional systems, the large scale web databases or from Ethereum, is that they design their contracts as a **monolithic applications** with their own variable length storage.*

> :globe_with_meridians: *For instance, in Ethereum, the token is implemented as a banking ledger, where you have a single smart contract that contains the list of accounts. And then if you have a million users, this list will be a million rows long. And there will be an entry for each user saying what is the address of the user. And what is their balance. And then designing such contracts is pretty easy and straightforward. You have atomic transactions, you deduct from one balance and add to another. And it's very easy to design and see how the system would work. 
**However, this model will not scale in TON because you would put this long list in a single contract and you will immediately run into issues.***

1. <u> :one: First, you will have progressively growing cost of rent for all of this data that you store in your contract. </u> :money_with_wings:
2. <u> :two: Second, every time the user comes along and wants to make an operation with this contract, they will have to pay the larger and larger transaction fees because they will be operating on the larger amount of data in the contract. </u> :sunrise_over_mountains:

## Rules.

*To help you, give you a guideline on how to properly design multi-user and large scale applications and TON, there are some rules.  :newspaper:*

> 1. :exclamation: *Avoid having variable length data.*
> 2. :exclamation: *If you have to have a list, then at least make it short and bounded, like statically defined.*
> 3. :exclamation: *If you need to have a list and it should be dynamically growing, then at least make sure that this contract is owned by a single user and they have the exclusive right to control its storage.*
> <u> A simple example are the records in the TON DNS items. </u>

## Blockchcain as an array.

*The scalable way to make large lists or dictionaries of data in TON is to use blockchain itself as an array. Here's a simple example.*

> <u>*You could imagine a system where you have multiple contracts that are connected :pushpin: with each other, and every user is taking care of their own individual contract. And you don't have to store the entire list inside your central part of the application.*</u>

> :gem: **The most popular example of implementing this idea is the design of tokens in TON ecosystem.** 
Balances of individual users are spread in so-called Jetton wallets.
> :fire: **What's cool about that is that the operations between two Jeton wallets in one place do not interfere with operations of the other two in some other place, and they could be spread among the separate shardchains and not create any bottleneck.**
