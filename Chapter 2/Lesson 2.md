# Messages types and computation phases

###### tags: `Chapter 2`

*Let's dive into how message is processed and what really is a transaction.*

> :love_letter: In order for a contract to change its state, it must receive a message. It's called an ***incoming message***.

> ##### There are two types of <u>incoming messages</u>:
> :white_circle: - *External messages.*
> :large_blue_circle: - *Internal messages.*

## External messages.

> ***External message*** - string of data that comes from nowhere from the perspective of a blockchain.
> - This data is not :id: authenticated by itself.
> - It doesn't have any money attached to it in form of :gem: Toncoins.
> - It can really contain anything that the author of a contract wants.

> :exclamation: ***It's the job of a contract to make sense of this data and parse it right inside its code.***


## Internal messages.

> ***Internal messages*** - those that are sent by contracts to other contracts. :busts_in_silhouette:
> ##### These messages have a little bit more rich structure:
> - Internal messages can carry the balances. :money_with_wings:
> - Those messages are securely authenticated by the address of a sending contract. :key:
> ***The entire architecture of TON guarantees the contracts that this address of a sender is correct.***


## Handling messages and opcodes.

*Once the contract received the message, <u>it could have two separate handlers for internal and external ones.</u>* 
Both kinds of messages will have the arbitrary :mount_fuji: payload data that is designed by the author of a contract. If the contract wants to process different kinds of messages, then it will use something that we call ***opcodes***.

> <u>***Opcodes (operation codes)*** :1234: - prefix of four bytes in the custom data in the message to indicate the type of the operation that the contract should support. </u>

## Transactions.

> :question: What is a transaction?

> <u> ***Transaction*** :money_with_wings: - complex of changes to the state of the contract. </u>
> :exclamation: *Message is not a transaction, it's just an input to a transaction!*

##### :question: <u> What does the transaction do? </u>

> *1. :arrow_forward: It changes the state of the contract.
> 2. :clipboard: It creates the list of outgoing actions.*

## Transaction phases. 

###### <u> There are five phases :five: that the transaction goes through: </u>


> 1. **Storage Phase** :ledger: 
*Deducts storage fees based on bytes stored by the contract since the last transaction. If there are insufficient funds, the contract transitions to a frozen state, preserving its state for a limited time.*


> 2. **Credit Phase** :bank:
*This is where the coins attached to incoming message get credited to the contract.*

> 3. **Computation Phase** :computer:
*This is where your program comes to life. The TVM executes the code and verifies each operation and also keeps track of gas usage.*

> 4. **Action Phase** :running:
*Most important action in this phase is the new state of a contract. Your contract may at the end of the execution or at any intermediate step create new state and new storage for itself and this will be recorded after successful execution of a contract. There are other actions in the list and those actions are outgoing messages.*

> 5. **Bounce Phase** :8ball:
*This happens if the contract failed and the incoming message had a flag saying I'm a bounceable message. It means that at this phase if there is any failure and there's any money left from the incoming message, then the contract would create the outgoing message back to the sender to bounce the money back. This is a safety feature that allows people to get the most of the funds back in case there is any error or any failure inside the contract.*

## Conclusion.

##### Let's make the summary:

- The contract receives an incoming message that could be *internal or external*.
- *Internal message*s can carry money and can be authenticated by the address of the sending contract, while *external messages* are not authenticated by themselves at all and it's the job of a contract to create the authentication. 

> - The execution of transaction goes through five stages.  
>    - Storage phase to charge the rent. 
>    - Credit phase when the incoming coins get credited to the balance. 
>    - Computation phase where your code gets executed.
>    - Action phase when the storage is updated and outgoing messages are routed.
>    - Bounce phase when the contract deals with the failures and sends back the coins to the sender.
