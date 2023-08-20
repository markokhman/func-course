# Blockchains, accounts, transactions in TON

###### tags: `Chapter 1`

## Blockchains and State Transitions
TON's blockchain serves as a ledger for state transitions. Smart contracts, referred to as accounts, possess their own unique addresses, storage, TON coin balances, and program code. Developers have flexibility in designing contract logic using the TVM virtual machine.

## Guarantees and Limitations 
Blockchain provides guarantees, such as verifying addresses and cryptographic assurance of code behind addresses. However, network speed is not guaranteed, and developers should focus on contract consistency and interactions.

## Messages and Transactions
Messages facilitate communication between contracts, containing coins and data. Transactions record contract activity. 

## Transaction Process
The blockchain structure changes only through external signals. Users initiate changes by sending messages to contracts. Validators process these messages and apply them to contracts.

## Double Spends and Consensus
Consensus prevents double spending by validators. Proof of stake is used, with validators putting up security deposits. Misbehavior leads to penalties.

## Validator Management and Sharding
Validator groups agree on transactions and can be split into subgroups as the network grows. Sharding is applied to reduce load.

## Two-Tier Blockchain System
Validator groups record account states on a non-shardable master chain. This division leads to delays in transaction processing and commitment.

## Delays and Commitment Process
Committing transactions takes around 10 to 12 seconds, with chains of messages taking longer. Initial transactions predict subsequent results, reducing concern about confirming each step.


## Conclusion 
In conclusion, TON employs a blockchain architecture where each contract is an isolated account with data and code. Consensus mechanisms prevent issues like double-spending. Despite some delays, transactions are confirmed relatively quickly.
