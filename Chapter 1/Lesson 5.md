# Chapter 1. Lesson 5. Breakdown the logic of Jetton

(DRAFT)

Jetton is a standard for fungible tokens that scales. Unlike Ethereum's ERC-20, each token is represented not by a single contract (with a long list of accounts and balances), but by a collection of per-user contracts that track per-user balances.

It has a clever mechanism to authenticate transfer of tokens: each time a token is transferred, another instance checks message sender against its own code. Contracts cannot see each other's state: they can only receive messages and trust the address of the message sender (this is securely provided by TON consensus mechanism). 

Tokens verify message sender by checking that the message arrived from the contract with the "same DNA" where the transferred quantity was correctly decremented. Token takes its code, the sender's "owner" field and reconstructs the contract address. If that address matches the message sender - it can be trusted.



