# Tokens

###### tags: `Chapter 1`

In this lesson, we will talk about tokens.

> :tada: An important aspect of TON platform is that you can create your own custom assets!

*This means really all sorts of things of value that you are going to transfer in your application:*
- Currencies
- Stablecoins
- Receipts
- Any types of tickets
- Any financial contract

> :green_book: ***Tokenization*** – process of breaking down the value stored in the system into transferable chunks.

:eyes: Let's take a look at what kind of tokens exist in TON. 

##### First of all, the tokens could be:
- fungible 
- non-fungible

*What does it mean?*

## Non-fungible tokens.

> ***NFTs (Non-fungible tokens)*** - tokens that satisfy following properties:
> - Unique
> - Have an owner 
> - Cannot be split or merged and freely interchanged

##### Non-fungible tokens implement a lot of important features in the TON ecosystem:
- TON DNS records
- Telegram usernames

## Fungible tokens.

*The fungible tokens add one more dimension – they also have multiple units that could be interchangeably transferred between the users.*

##### In Ton, the fungible tokens are used to implement a lot of things:
- Currencies
- Cryptocurrencies
- Shares
- Voting rights

> :orange_book: **In TON fungible tokens are implemented in a scalable manner.**

*In TON you don't have the single contract that keeps a track of all the accounts that own portions of the token or the units of the token. Instead, there is a multitude of independent contracts with the same code that are called token balances or token wallets. And every user's wallet communicates directly with their own token wallet.*

## Scalability of tokens

> **What does scalability of tokens mean?**
> *It means that while :dancers: two users are making a transfer of one token between themselves, this transfer doesn't interfere with the transactions of some other :two_men_holding_hands: two users who are doing the transfer of the same tokens on some other end of the network.*

>  :fire: In TON, all these custom assets that you create inherit the full unlimited scalability of the network without any shortcuts.

## Conclusion

###### Based on the above, the following conclusions can be drawn:
- :money_with_wings: Fungible and non-fungible tokens allow you to implement all sorts of assets that you could transfer directly to the users.
- :gem: Together with contracts, tokens that are built with contracts become one of the building blocks in the arsenal of an app developer.
- :books: Understanding how the tokens work is important to make your system scalable and performant for all the users at any scale.
