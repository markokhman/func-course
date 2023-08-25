
# Blockchain design principle and three generations of blockchains

###### tags: `Chapter 1`

*This lesson explores the historical evolution of blockchain design, focusing on three generations of blockchains: Bitcoin, Ethereum, and TON.*

## Bitcoin Blockchain

Bitcoin was the **first blockchain, designed to address censorship-resistant digital cash transfers**. It utilized a ledger with accounts identified by public keys, allowing the transfer of coins between accounts. Transactions are authorized using cryptographic signatures. **Bitcoin's structure included multi-input/output transactions and chain consensus, maintained by Proof-of-Work**. A scripting language extension enabled complex control over transactions, though limited in scope. Bitcoin's privacy measures included generating new account addresses for transfers.

## Ethereum Blockchain

Ethereum, a **second-generation blockchain, broadened the scope by introducing composable contracts and a flexible account structure**. It allowed users to execute arbitrary functions on the blockchain by paying for execution costs. This flexibility enabled decentralized finance applications. However, Ethereum's richness in functionality posed scalability challenges due to the architecture's limitations.

> ##### Summarizing the above about Ethereum, the following things can be highlighted:
> - Ethereum tries to generalize the idea of Bitcoin and make it more flexible for developers.
> - Every account in Ethereum may not only have the predicate code, but may also have arbitrary internal storage.
> - Your transitions no longer just transfer from one account to another.
> - The accounts may communicate with each other and they can send messages synchronously to another account, just like calling functions in a single application.
> Ethereum ideas are compatible with many known tools and programming paradigms. 

> :exclamation: **Ethereum's architecture is very flexible for developers but completely unscalable in principle, because it exposes the developer and all the contracts to global storage and global state of the system.**

## TON Blockchain

TON, a **third-generation blockchain, introduced limitations to achieve scalability**. Contracts in TON had localized visibility and communicated through messages, unlocking scalability potential. Proof-of-Stake replaced Proof-of-Work for consensus, allowing separate validator groups and efficient message routing. TON implemented precise cost controls, requiring payments for execution, data storage, and message routing, ensuring scalability and mitigating denial-of-service risks.

> ##### The following facts are true about TON:
> - TON's idea is to figure out how to provide the developers infinite flexibility and scalability.
> - Contracts in TON are not allowed to see the global state, they can only see their own state.
> - The only way that contracts can communicate with other contracts would be asynchronous message passing.
> - Every time you make a transaction in a contract, this transaction is 100% independent from another transaction on a separate contract, and those could be processed in any order or independently.
> - In TON all contracts could be sharded, and also they could communicate with each other, and those messages are routed by the system.

## Conclusion

*In summary, the evolution of blockchains progressed from Bitcoin's simple yet impactful design to Ethereum's enhanced functionality, which encountered scalability limitations. TON, as a third-generation blockchain, addressed these limitations through innovative constraints, scalable features, and meticulous cost controls. **This novel approach paved the way for infinite scaling possibilities while ensuring security and efficiency in blockchain operations.***
