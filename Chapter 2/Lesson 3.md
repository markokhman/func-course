# Authorization of messages in contracts

###### tags: `Chapter 2`

This lesson discusses various authorization methods for messages.


## Overview of all authentication types

1. **Authentication with a signature.** 
> :exclamation: Any event in the TON blockchain should start with an external message.
*Whenever the external message comes in wallet, it reads off 64 bytes of data :six::four:, <u>that is the signature for the rest of the message</u>, verifies the signature with its public key, and then treats the rest of the message as the instructions to send other messages to other contracts internally inside the blockchain.*

2. **Authentication of a message sender.**

> *All the internal messages in TON are identified by a message sender that is guaranteed to be correct and secure :unlock: by the TON protocol. Every time a contract receives an internal message, it knows for sure from which other contract the message was received. And this is immensely more cheap and powerful than checking the signatures.*

3. **DNA check for trusted code.** :raised_hands:

> *Since the addresses in the TON ecosystem are not simply unique identifiers of the contracts, but they're also cryptographically secure hashes of the contract code and data, they don't change when the data changes. Since those addresses are cryptographic commitments to this code, you could verify what kind of code is talking on the other end by checking the message sender.*

4. **Lack of authentication for censorship resistance.**

> :question: *Isn't it insecure to not authenticate the messages? In certain situations, this is where the security actually lies. You can deal with a system that must be censorship resistant like decentralized staking pool*
