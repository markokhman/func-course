(DRAFT)

TBD: internal VS external

- internal - cross-contract / authenticated via "message sender"
- external - from humans - authenticated via signatures or non-authenticated

TBD: message opcodes

- convention to dispatch different actions

TBD: phases: receive / compute / action

- rent, gas limit
- TVM instantiation and gas payment
- new state, outgoing messages


-----------

Chapter Two, lesson one, the TVM and contract storage for the data layout and talk. So in the first chapter, in lesson four, we talked a little bit about the anatomy of a contract. And in this chapter we'll deep dig a little bit deeper to see how exactly the contract is executed. And how its state is laid out. This whole chapter will be spent on talking about different aspects of contracts, design and contract functionality at the low level of the blockchain. And at this first lesson, we'll talk about the execution and the memory layout. 
 

**Introduction to TON Contracts**

Now, let's remember what the contract really is. Contract TON is an entity that has data, it has code and has a balance of total coins. And all of these are encapsulated in a contract instance. And all the code that is executed inside this instance, it can only see this its own state, and a few like network wide configuration parameters, but it cannot see the state of any other contract. So how do contracts communicate with each other in the real world? So this is done with messages. And we'll talk about the message processing phase in a separate lesson. But for today, it's sufficient to say that the contract is activated when it receives the incoming message. There are a couple of sorts of messages. But what's important is that it receives some structured amount of data as an incoming message, it can parse it, like open it up, see its contents and decide what to do. And what contracts really could do is to do any arbitrary computation with the incoming data and its own state. And it can save this updates to its state and also issue outgoing messages that it will send to other contracts in the network. So the incoming message is only one at a time. So every message has one specific recipient and is processed atomically as an atomic transaction. But the contract may emit multiple outgoing messages.

**Understanding TVM**

Now, what is TVM? What is the role of the virtual machine here? So the neat thing is that TVM, the term virtual machine, is not a global instance, that sees the state of the entire system. Instead, TVM is a pretty lightweight bytecode machine that is instantiated, every time the contract needs to process a message. So the input to the TVM (like the model is pretty purely functional) is the state of the contract, configuration parameters and the incoming message. It picks up the code from the contract executes, the code does all the changes necessary that are not even done in place, but instead admitted as results or actions of the TVM execution. So if you execute the contract this way, then you can do this, even locally, it's very easy to locally test it and audit. And all the changes to the system will be reflected in this list of actions that TVM emits. And normally, those include the new state of a contract, and zero or some number of outgoing messages. There are some other special types of factors, but you normally don't have to be concerned about that. 

**The mechanism of TVM operation**

So what does TVM look like? So it's a bytecode stick machine that has a pretty standard set of operations for arbitrary computations. It has basic primitive types, like integers, and it can operate on the TON data types. So the interesting part is that in TON (the memory layout of TON) you don't have arrays or strings. You have one integer type that occupies 257 bits, so it's a 256 bit integer with a sign and a flag with this, like a failed arithmetic operation or not, like not a number flag. And the other type is a cell. So this is very important because the cell is not just the type inside the TVM, the cell is a data structure that is used throughout TON blockchain, all over the system. So all the messages, all the layout of folder data structures in TON, like built in or within the contracts, they're all built on cells, the entire state of the blockchain with all the master chain and work chain, all the short chains, all the messages, all the transactions, they're all built out of cells.

So once you understand what it is, then you will see that this is the building block for everything that happens around. 


**Limitations of TVM**

And it's important to understand the kind of trade offs and limitations of this datatype. So cell is the thing that holds some number of bits. So it's not advice, it's measured in bits, from none to 1023 bits of data and no more than that. And it may contain up to 4 references to other cells. So your only datatype is this little buffer of data that is from zero to 1023 bits long, you can't refer to more than that. And you're allowed to have none or some number up to four references to other cells. So if you have a lot of data, you can peck it out in this kind of like tree structure or change way into cells, if you wish. So why is this design like that? Surprisingly, this is the only data type that exists in TON and if you look at the TVM, there is no way to allocate memory, like in Ethereum.  You can’t just say “I want that much of space”, “I want one kilobyte of space” or something like that. You could only work with cells. And since everything is built on cells, you have a few interesting properties. 

**Hashing**

So first, hashing. Merkle hashing of these data structures comes for free and it's well defined in the system. So the hash of a cell is a hash of its contents plus concatenation of older hashes (what goes to the hashes of all the inner cells). And this naturally allows you to do Merkle trees of anything. And this is a number one reason to even design like this, because now you can localise the entire state of the blockchain and make logarithmically sized proofs about anything that happens in it. The second reason is that this kind of hashing algorithm allows you to implement effective deduplication and compression. So you could have some of the data pruned from a contract either it's your application choice, or it's like a system level pruning of the unused storage. And structurally, all the logic of the blockchain would still work and request missing data on the fly. 
And actually, cells have a few flags to kind of signal whether this is approved, sell or not, but those flags don't affect the hashing, the only affect whether the contract is available or not. So, using the cells, you can build up the contract state. So the contract `seizes (8.56) effectively` just one cell. And if you want to store more data than 1023 Bits, then you have to pack it into inner cells. 

You would typically see the TLB schemes that described contract layouts or message layouts. And a TLB is pretty powerful language that also describes various types on a bit level. And it even has a syntax for showing you explicitly whether certain data goes into a separate cell, like sub cell reference. But what's important is that the TLB doesn't know about the limits. So it's important for the developer or for the level of the programming language that you use to keep track of whether you actually fit in the limits. So if you try to actually write more bits than available in the cell, that operation would fail at the serialisation time. `And likewise, for no more nested cells than for fourth time. (10.06) `



**Unpacking and it’s conditions** 

Now, I mentioned that this is pretty limiting, so if you want to create a list of things, you have to kind of invent it using cells and this is done by design. So, it's not comfortable to do lists or dictionaries like this,  because of this requirement to allow localised representation and logarithmic proofs, and to enforce this, so that people don't run around creating long chains of cell data structures, TVM implements dramatic costs of unpacking inner cells. This means that every time you want to dig into the sub cells, you have to pay an extra fee. And this fee reflects the need for the system to verify the hash and acquire this nested data. So the more or less data you get to transmit in presented store in the blockchain, the more cost you incur on the system.

And if you try to make a long chain of the cells, then you'd be going to linearly growing price for carry overs around. 

**Methods that allow you to lessen the cost**

They're a few things that help you with this. First of all, the standard library in FunC comes with a HashMap primitive that is built out of cells. So in fact, the HashMap allows you to package up your key value pairs in a binary tree, or close to binary tree depending on the distribution of keys, that kind of puts data in all this kind of trees, and sub trees of sets. And standard library gives you nice tools to work with those dictionaries, so you don't have to think about the cells when you do that. But there are other considerations regarding scalability and denial of service attacks, as we will discuss in this chapter in the lesson four and five, that are very important to consider, that put a limit on how much space you actually want to claim using HashMaps. So there's some legitimate use cases for HashMaps, but you shouldn't expect that you could just put the whole banking ledger inside such a thing. 

**Summary** 

`So, to recap, contracts of code and data TVM executes the contracts, TVM(12.45)` is pretty flexible, you can modify the data, how you please, you can also modify even the code, you can replace the code of the contract, can emit the messages, and everything in the TON is encoded using those cells. And at the TVM level, you have cells and integers as your primitive data types. 

**Other data types** 

There is also another type called continuation. This is effectively just chunk of code that is also represented as a cell that can be executed. So it's like a type marker. And this continuation is used to present functions, so you can call them and store them around. And there's another type called bag of cells, that allows you to transmit bunch of cells together in one package. And this bag of cells takes care of deduplication. So every time you transfer a message, and you have some new pieces of data that is duplicated, or like some cells reference the same content, this bag of cells automatically deduplicates this information. So it's sort of similar to how Git compresses data in the repository. So it deduplicate repeated files. So you have the impression that things are semantically duplicated all over the place and you have a complete snapshot or repository at every commit. So likewise, in TON, you may have copies of the thing in different stages, but under the hood, those duplicates will be optimised to it. So yeah, that's it for the structure.
