# Chapter 1, lesson 1
## The overview of the TON ecosystem

What is TON? It is a technology platform for building decentralized applications. TON is not just a blockchain, it's the entire ecosystem of various entities. In TON, there are different parties: there are users who hold assets that are existing on TON; there are validators who operate the core consensus to make the blockchain hold together and, also, there are app developers who provide all the services that use TON blockchain as their platform.

### Custodial vs non-custodial services

Then there are different categories of apps and services. The most important distinction is between the custodial and non-custodial services. Here we have a couple of examples:
Custodial services are those that are operated by a party that bears responsibility for the deposited funds. Hence, online exchanges are typically custodial services, because they need to take the coins or tokens on the balance of their own wallets, to guarantee trades. But this also creates risks, because exchanges could be hacked, and their funds could be stolen and impossible to recover.
As for non-custodial services, they are such that they don't hold users' funds and users have direct personal control over them. Such services keep all the assets in the hands of the user. So, it becomes harder to pull them together. And for this, you need to do some blockchain smart contract mechanics that allow people to safely use the assets in one place, but this place would not be a custodial service, it would be sort of a smart contract, because it provides strong guarantees of security.

### Custodial vs Non-Custodial Wallets

There is also division into custodial and non-custodial wallets. Most popular in any cryptocurrency are non-custodial wallets, where the user holds the secret key that is used to authorize transactions. And they're responsible for not losing this key and authorize various actions on the blockchain with that key. So, most of the time, the funds are on their wallet, and the users can send them to exchange or send them to some decentralized application. They do have this full control and no other party could just pull the money from this wallet, like from a credit card.

### Centralized vs decentralized apps

There exists another distinction between the centralized and decentralized apps. What does it mean? A centralized app is the app that effectively has a server. While a decentralized app doesn't require a server, it runs on the blockchain’s smart contract. Let’s have a look at an example:
The wallet is usually plus or minus a centralized app, because it uses some server to index the blockchain show you the balances. But it still could be non-custodial, cause it's not holding your keys and can't steal your money. It just provides the utility of the current status of your balance.
Decentralized exchange is one of the examples of a decentralized app. It has a frontend -- some UI that helps you to fill in the form. But the same app may have multiple frontends. And you don't really depend on a server that provides this frontend or any like application. Next, this frontend goes directly to the blockchain to send transactions or get the status from the blockchain. And the backend is not a server, but the blockchain itself. It means that your data is stored in a smart contract. And execution over this data is processed by the entire network instead of your private server. 

### Exploring Entities Built on Blockchain: Smart Contracts, DEXs, and Tokens

And finally, there are different categories of entities that you can build using smart contracts on a blockchain. It's not only about the applications that could have the logic which is pretty obvious: you can have a little program that implements trading logic, for instance, and these are called DEXs -- decentralized exchanges, but also things like tokens. Those are
are assets on the blockchain and they're also modeled as contracts. So, if you can look under the hood, everything's just smart contracts on the blockchain, but if you zoom out, they have very different meaning and role, so there are different entities: apps, tokens, auctions, domain names.

### Understanding the Diversity of Tokens and Smart Contracts on the Blockchain

There is a wide diversity of tokens in TON. They could be applied as currencies, as stable coins, fully fungible and exchangeable. They could be also collectibles or utility tokens. So, for instance, the TON DNS record that holds globally recognized name “.TON” with various user defined attributes. That's a non-fungible token that could be transferred and sold. It is implemented using smart contract language.
Also smart contracts could act as utilities within a larger system. In a complex application you may use multiple interconnected smart contracts that manage large amount of data. You are likely to have a central smart contract that provides entry point to your application. Around that contract you would have satellite contracts: for instance, tokens and receipts for each user, or separate contracts for intermediate computation states.

### Conclusion

In TON ecosystem you have all these various entities on the blockchain: you have wallets, exchanges, traders and consumers who interact using smart contracts and application software around them. Later in this course you will see chapters on things like authorized smart-contracts or like TON connect that allows the communication between the wallet and the decentralized application. We will also discuss nuances of communicating with decentralized apps and centralized apps, and how to build axillary features for each of these entities.
