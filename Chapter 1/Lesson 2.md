# Chapter 1. Lesson 2. Understanding the concept of account.

### What are accounts?

Account in TON is also known as a "contract". It contains three key ingredients:

* Code
* Data
* Coins

The job of the contract is to protect its state (data and coins) while implementing some logic in deterministic and useful manner.

TON protocol has very little logic baked into its implementation. The idea is that built-in logic is hard to upgrade, while contract code can be created and improved by anyone. TON's core job is to correctly execute any custom code and enforce some invariants (verify transfers of coins + collect transaction fees).

### What can I do with contracts?

Contract can implement a wide range of concepts:

* Wallet accounts
* Multiparty contracts
* Tokens
* DNS records
* Exchanges
* Liquidity pools
* Price oracles

Together, all the contracts form the _TON ecosystem_. People can create fully decentralized and immutable contracts and build other contracts that fully trust them.

### Contract address

Each contract is identified by a hash of its code + initial data and looks like this:

```
EQAhE3sLxHZpsyZ_HecMuwzvXHKLjYx4kEUehhOy2JmCcHCT
```

The address also contains the ID of the workchain: some accounts may exist on the masterchain, but most sit on the "basechain". Sharding does not affect account identifiers: the first bits of the address determine which shard (that is, which validator subgroup) manages transactions on that contract.

### Interacting with contracts

To interact with the contract one need to send it a message. Messages are usually formatted using TL-B scheme that precisely specifies layout and semantics of the data.

A contract reacts to an incoming message (one per transaction), updates its state and potentially emits a few outgoing messages to other contracts.

### Encapsulation and mutability

A contract is fully self-contained entity that does not "see" anything outside its own storage and code. Each transaction replaces one state of the contract with another. This model helps with audit and testing, and enables truly scalable execution.
