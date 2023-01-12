# Chapter 3. Lesson 1.

### **Intro to Chapter 3**

In this chapter we are going to get very practical and will actually breakdown the whole cycle of TON smart-contract development, go through each step of it together and the end result would be a ready-made local setup for smartcontract programming, a written FunC code of the contract, tests for our contract an actual deployed contract.

Let’s get straight to the point.

### **TON smartcontract is like a sattelite.**

The best allegory that i was able to come up with to explain the lifecycle of the TON smart-contract is following. You can think of a TON smart-contract is a satellite, that is launched to the orbit of the Earth.

The satellite is flying around the Earth, interacting with other satellites, able to accept information from the earth, process it and send some results. But before we actually launch it to the space there is a few stages it has to go through.

- There is a base laboratory where it is getting assembled. [Local setup]

- There is a certain documented protocol that defines all the possible commands that could be processed by our satellite. [TLB Scheme]

- Each of the behaviour scenarios are tested while the satellite is still being in laboratory. [Local tests]

- After that it’s put into a simulated environment very similar to what is awaiting for it in Space, this is allowing to test one more time the expected behaviour in response to predefined commands as well as interaction with other subjects in space and on the Earth. [Testnet and after-deploy onchain tests]

- How do we deliver the satellite to the orbit? Using rockets. Rockets are bringing the satellite to the orbit and leave it there, falling off after the successfull finish of their mission. [Mainnet production deployment]

From this moment the satellite is on it’s own in the Space working with everything it has preprogrammed inside.

**Explaining how the TON smartcontract development lifecycle relates to the sattelite example.**

When we are writing a smart-contract we are using a similar cycle.

**1. We prepare our local setup, that will enable us to bring our satellite/smartcontract through every stage mentioned above.**

The first key part of our local setup will be compiler. The actual smartcontract on the TON blockchain is stored and executed as a binary code. But we want to program logic with something understandable for human, so the language we are using to program TON contract is FunC.

The path between FunC and bytecode is as following: FunC is compiled into Fift assembler code, which generates corresponding bytecode for the TON Virtual Machine (the Space, if we reference to our satellite example).

For us, the essense of the Fift assembler code is not very important and we will treat it just as an under the hood intermediate state of our smartcontract code. We give this away to our compiler.

Our code -> FunC -> Fift -> BOC (Bytecode)

> I believe the part about explaining how bytecode is stored and executed should be explained in chapter 1 and 2 (a tree of cells etc.)

At the moment, the best and mainstream way to operate the compiler is using TypeScript language. TypeScript has nothing to do with the TVM. Think of it as something that stays on the Earth once the satellite is launched.

The compiler we are going to use is [@ton-community/func-js](https://github.com/ton-community/func-js)
Internally this package uses both FunC compiler and Fift interpreter combined to single lib compiled to WebAssembly (WASM).

**2. We use a TL-B schema to describe the commands that our smartcontract is able to process**

TL-B (Type Language - Binary) serves to describe the type system, constructors and existing functions. Here is an example of a possible TL-B document:

![alt text for screen readers](https://ton.org/docs/img/docs/tlb.drawio.svg "TL-B Schema")

Usually we are writing our TL-B document once some of the basic functionality is ready and then we keep updating it all the way until launching the contract. We are going to get closer to it, once we write our first FunC code.

**3. The sweetes part. We write FunC code.**

You've already seen some FunC code in the first two chapters. However in this FunC is a domain-specific, C-like, statically typed language.

**4. We test our FunC code behaviour locally.**

Then we are using TypeScript again to write test logic. On this step we are simulating TVM machine locally, send data into the simulated contract, analyse the output, repeating that until we get desired results. Once ready for deployment - we are writing a TypeScript code that would actually deploy our smart-contract to the blockchain. This is our rockets.

**5. We deploy our code to testnet.**

The process of deployment of smart-contracts is very interesting in TON. What makes it so interesting? As you could see from the previous lessons - we can calculate the address that smart-contract will have before we even deploy it to the network. To make this, everything we need is to know the initial state of data for the contract and it’s actual code. Once we know the address - we are sending a message with initial state and code to this address. That simple. It could be an internal message or external message.

So our deploy script would be as simple as calculating the address and sending a message with initial state and code to this address.

You might have noticed, that by sending message we have to spend some actual money, so we don’t want to spend lot’s of real money while deploying and testing our contract on-chain.

This is why we have a copy of the real network that i used only for test purposes. It’s free to get coins that are valid on this network and we call this network testnet. Before our contract is fully ready for production - we will only deploy it to the testnet.

**6. The launch. We deploy our code to mainnet.**

Deploying the smartcontract to the mainnet is pretty much the same as with the testnet, but we are deploying our contract by sending messages with real TON coins, our messages get on the main blockchain and our contract is available for all the TON users.

It is a very interesting process. Sometimes it's hard to find answers, but I encourage you to not give up and throught the path of this chapter I will make sure that you:

- Have your local "laboratory" of full cycle for creating and "launching" your smartcontracts
- You will understand the basics of coding a FunC contract
- You know where to search for answers, once your contract requires more then we cover in examples

Let's go for it!
