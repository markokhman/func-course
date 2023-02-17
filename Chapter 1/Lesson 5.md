# Chapter 1, lesson 5
## Tokens

Important aspects of the TON platform is that it allows you to create custom types of assets. This means not only custom currencies, but all sorts of things of value that can be transferred and transformed and passed around not as applications but as things of value. Collectively they are called tokens. And this is a technical term, that means a lot of things. Basically, everything of value is token, splitting the value of certain systems into transferable chunks is referred as *tokenization*.

### Fungible vs non-fungible token
What types of tokens could we imagine? First of all, there's a well-known division into **fungible** and **non-fungible** tokens or NFT's. 
First of all, the main difference is that non-fungible tokens are unique. They have a property called owner, but they cannot be sliced down into equally interchangeable parts. So unique tokens are *collectibles*. Examples:
* Tickets
* TON DNS records 
* Usernames 
* Financial contracts
* Subscription system money collecting object

By being a token, it permits its owner to change the owner, rotate the keys, and sell the business.

As for fungible tokens, they are those which don't have just the owner, they do have a counter and number of items. The obvious example of such are currencies. They are:
* Stable coins
* Cryptocurrencies
* Monopoly money

All of these are examples of fungible tokens. Further in the course we will talk about the concept of their work in a more detailed way. 
Fungible tokens, they have the owner, and the way they are implemented is that each token has a contract for each owner. Instead of switching the owners instead, in those instances they *transfer messages* between each other to update the balances. It means that each contract holds the balance for each owner. In case you want to transfer a certain amount, then it simply decrements this amount in its own storage, and sends a message to its sibling contract to increment its counter.

### Tokens in TON
Concerning both fungible tokens and non-fungible tokens, they're all scalable in TON. This is an important aspect of TON. Non-fungible tokens and fungible tokens are implemented in a way that they can scale to billions of users. And transactions between them don't create bottlenecks in some specific place. 

Let's say you have a system where the balances may have different states. So you may have money that is temporarily unavailable, but you want to record that the user has deposited it. The scalable way is to implement extra tokens. You may have your primary token – that is what your system is designed around your main stable coin or cryptocurrency; auxiliary tokens that represent those in a different, for example temporary state; smart contracts that would convert one form or another. 
Hence, from the users' perspective, these auxiliary tokens are not really things of value, they probably don't even have to see them as different assets in their wallets, instead, they would probably just see their normal tokens in just different states.

### Conclusion

As seen, even the token is still in some cases could be a low level building block for a higher level of concept. So on one hand, you have this low level notion of a contract that allows you to implement the token and then it may turn out that the token itself is a low level building block for some other thing we could do to implement tokens in the different states. 
This is something to be kept in mind while designing applications in TON.
