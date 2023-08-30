# Review of classic business problems

###### tags: `Chapter 2`

*Here we will go through three examples to see how the principles of contract design apply to various applications and how TON platform helps you to build scalable and secure applications.*

## Tokens.

<u>In Ethereum or even non-blockchain implementations, the tokens would be implemented as a **simple ledger of the accounts**. </u>

*You have a list, the program controls this list of accounts and each item in the list is effectively the address of the participant and their balance. This is pretty simple but unfortunately it doesn't scale in the blockchain context because your smart contract has to continuously grow with the number of users and becomes more expensive for each user to interact with.*

> :question: **What is the TON approach?**

> **In TON the tokens are implemented as distinct contracts.**
*One contract per user plus a separate minter contract that provides the interface for creating new units of the token.*

The contracts per user are called **Jetton wallets** and the job of those Jetton wallets is to **hold the balance of a token for each individual user**. Whenever the user wants to transfer a token from one account to another :money_with_wings:, they send first of all an external message to their wallet, then this wallet unwraps this external message and sends through the internal message to this user's Jetton wallet saying <u>"Please, send the money to a certain address"</u>. Then Jetton wallet decrements their balance by the necessary amount and sends the message to its sibling contract that has exactly the same code but a different owner and this message says <u>"Please,increment your balance with the same amount</u>.

> :bread: ***DNA-check is used here because the code of the Jettons is the same.***

## Multi-signature contract.

> :book: *The concept of a multi-signature contract is exemplified in the context of TON, where multiple parties collectively authorize actions by tokenizing user-initiated requests.*

Instead of directly processing messages, **users receive unique tokens encapsulating their requests and votes**. These tokens, owned by users, simplify tracking and prevent malicious actors from overwhelming the contract. Honest users gather votes on temporary request tokens, and once a threshold is met, the multi-signature contract executes the action after verifying the request's authenticity. This tokenization streamlines the process and focuses on the temporary state of the system rather than individual contract shares or values.

## Subscription payments.

> :eyes: *Since the wallet version 4, the wallets support the plugins that enable people to create subscription payments.*

This is a very powerful feature and it scales relatively well because the user controls this list of plugins. But we could imagine a **better way to do that**.

> - :fire: <u> Cool idea is that instead of listing particular plugin addresses, you could list distinct code implementations of those plugins. And if they share the same code, there would be just one record for any number of subscription plugins that you have. <u> 
