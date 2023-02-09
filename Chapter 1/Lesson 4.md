# Chapter 1. Lesson 4. Breakdown the logic of NFT (Non-fungible token).
 
(DRAFT)

### Usage

NFT originally came about as collectibles with pictures. But that are highly useful for encoding any kinds of receipts or tickets. E.g. a paid subscription could be an NFT. All the users pay coins to that contract, while the owner can withdraw the money from it on a regular basis. If the owner wants to rotate keys or sell their business, they simply transfer NFT to a new address.

### Mechanics

NFT is a contract with `owner` field and `init` flag.

NFT is a "unique" token. It is tied to its "minter", aka "collection". 

Transferring means changing the `owner` field.

### Definition of existence

Why init flag? Because in TON contracts are not "deployed"/"initialized" really. Anyone can send any message and deploy the contract code (if it's missing) at any time.
Init flag defines token as "existing" or "non-existing". When it receives initial message from its collection, it sets the flag on and starts to "exist". It can then respect transfer message and perform any other things.

