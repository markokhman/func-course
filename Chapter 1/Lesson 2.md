# Blockchains, accounts, transactions in TON

###### tags: `Chapter 1`

## Blockchains and state transitions.
TON's blockchain serves as a ledger for state transitions. It's a ledger of changes in the state of arbitrary accounts.

> Every smart contract is also called an account.

You should know the following things about **accounts**:
- has its own storage
- has its own unique address
- store the balance of Toncoins
- store the program code 

##### Developers are provided with pretty much unlimited flexibility through the lifetime of the contract.
- They can change the data and they can even change the code of the contract
- They can transfer the coins around
- They can send messages to other contracts

However, there are justifiable limitations justified by the TON architecture:
> :eyes: Contracts have no visibility into anything outside themselves. By isolating contracts from each other, TON is infinitely scalable. 

> :globe_with_meridians: **TON smart contracts concept look very similar to computers on the internet**. In order to use some other contract state, the contract has to send an outgoing message and maybe later it will receive the desired reply. Like every computer is independent, they have their own data, they are programmed separately, and they have no idea when and how their messages will be processed by others. 


## Guarantees of blockchain.

There are some <u>*guarantees*</u> that the blockchain provides to you as a developer.

> :white_check_mark: You could verify the address of any incoming message and be sure that behind this address, there is a specific code that you may trust. 

> :white_check_mark: The network guarantees the delivery of messages, but it doesn't guarantee how long this will take.

:x: The chain of messages could spend thousands of shards and take multiple blocks to finalize the transaction, so relying on a certain delivery time isn't really accurate. 

## What are messages and what are transactions?

> :envelope: <u>*Message*</u> - the things that happens in between two contracts. It carries a little bit of coins and arbitrary data to a contract address.


> :gem: <u>Transaction</u> - activity on the contract including running contract code, updating contract state, and emitting new messages.


## Every contract has its own little blockchain.

The blockchain itself is just a data structure. To change it, there must be a signal from outside.

##### An example sequence of actions is roughly as follows:
1. User presses a button on the wallet.
2. The wallet creates an external message to a destination contract.
3. The wallet sends the message to the validators who apply it to the contract.

> :exclamation: Every contract has its own transactions, which means that every contract has its own little blockchain. And this is quite helpful, because **the network can then process and verify transactions completely independently from each other**. 

> :question: However, questions remain. How do we route these messages, and how do we prevent double spends?


## Double-spending and consensus.

> :dollar: <u>Double-spending</u> - the expenditure of the same digital currency twice or more to avail the multiple services.

Consensus prevents double spending by validators. *Proof-of-Stake* is used, with validators putting up security deposits. 

> :gun: Misbehavior leads to *penalties*.

> :paperclip: In TON, the set of validators can be **sharded**.

If the load on the system increases and the number of account grows, then the sets of validators could be split in subgroups. 
TON allows **sharding** to be pretty much unlimited down to individual contracts, depending on the load.


## Two-tier blockchain system in TON

In TON we have two-tier system:
- We have **masterchain**.
- We have **basechain**.

When all the validator groups achieve agreement on their parts of the blockchain, they record the state of the accounts on a central non-shardable blockchain called **masterchain**, where every validator participates.

The basechain is *virtual*, because it actually contains all of these various accounts that could be sharded infinitely down to each individual account and could be split and merged into the **shardchains**.

> :exclamation: Remember the following facts about masterchain!
> - It is **not shardable**.
> - It is **expensive in terms of fees**.
> - It **doesn't scale**.
> - It **contains only the configuration of the network and snapshots of all subchains from various validator groups**.

## Transaction processing latency

Committing transactions takes around 10 to 12 seconds (5-6 seconds to *basechain* and 5-6 seconds to *mastechain*), with chains of messages taking longer. Initial transactions predict subsequent results, reducing concern about confirming each step.


## Conclusion 
In conclusion, TON employs a blockchain architecture where each contract is an isolated account with data and code. Consensus mechanisms prevent issues like double-spending. Despite some delays, transactions are confirmed relatively quickly.
