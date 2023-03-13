# Chapter 3. Lesson 4.
###### tags: `Chapter 3`

We've written our first FunC contract, we know that it succesfully compiles. What's next?

In this lesson we will learn how to make sure our contract's code is actuall working as expected. Let me remind you our expectations to the code:

Our contract is supposed to save a sender address every time it receives a message and it should return the latest sender's address once we call the getter method.

Great, but how to we ensure that it is working? Well, we could deploy it to the blockchain (which we will eventually do very soon, in the next lesson), but TON has some tools, that allow us simulate certain behaviours locally.

This is done with help of `sandbox`. This [library](https://github.com/ton-community/sandbox) allows you to emulate arbitrary TON smart contracts, send messages to them and run get methods on them as if they were deployed on a real network. Because we are having our TypeScript "laboratory", we can create a sequence of tests with help of another library - `jest`. This way, we have a test suite that simulates all the important behaviours, with different input data and checks the results. This is the best approach to write & debug & fully test your contracts before launching them to the network.

### Preparing our test suite

First of all, assuming you are currently in the root of our project, let's install `sandbox`, `jest` and one more library we will need to interact with TON entities - `ton`:

```
yarn add @ton-community/sandbox jest ts-jest @types/jest ton --dev
```

We will also need to create a **jest.config.js** file in the root of our project for **jest** with following contents:

```
module.exports = {
    preset: 'ts-jest',
    testEnvironment: 'node',
};
```

Now let's create a new folder **test** with file **main.spec.ts**:

```
mkdir tests && cd tests && touch main.spec.ts
```

Let's set up our **main.spec.ts** file for writing our first tests:

```
describe("main.fc contract tests", () => {

  it("our first test", async () => {

  });

});
```

In case you've never written TypeScript tests, you should definitely  try this approach. While programming TON smart contracts you are going to spend more then half of your programmin time on writing tests. Remember our Space satellite example? Same here, we simulate every important case before we ever deploy the contract even to testnet.

Now try running command `yarn jest` in the root of your directory.
If you've installed everything properly (step-by-step with me) - you're supposed to see following:

```
 PASS  tests/main.spec.ts
  main.fc contract tests
    ✓ our first test (1 ms)
```

Great, let's immediately create another script running shortcut in our **package.json** file:

```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "test": "yarn jest"
  }
}
```

### Creating contract instance

In order to write our first test, we need to understand how we can create a TypeScript instance of our compiled contract with help of `sandbox`.

We've discussed before, that our contract code is stored as a Cell after being comiled. In our **build/main.compiled.json** file we have a hex representation of our Cell. Let's import it into our file with tests along with a Cell type from `ton-core`:

```
import { Cell } from "ton-core";
import { hex } from "../build/main.compiled.json";

describe("main.fc contract tests", () => {

  it("our first test", async () => {

  });

});

```

Now, to restore our hex and get a real Cell, we will use this command:
`const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0]`.

We create a Buffer from a hexadecimal string and pass it to the **.fromBoc** method.

Now let's take a look at **sandbox** [quick start guide](https://github.com/ton-community/sandbox). We see that before getting an instance of a contract, we have to get an instance of blockchain. 

Let's import a class Blockchain from `sandbox` library and call it's `.create()` method.


```
import { Cell } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { Blockchain } from "@ton-community/sandbox";

describe("main.fc contract tests", () => {
  it("our first test", async () => {
    const blockchain = await Blockchain.create();

  });
});
```

<!-- We see that we can import a class **SmartContract** and it has a method **.fromCell**, that is expecting to receive two Cells, namely **code** Cell and **data** Cell. -->

Now let's get ready to get an instance of contract to interact with. Sandbox's documentation states that **the recommended way to use it is to write wrappers for your contract using the `Contract` interface from `ton-core`**

What would it mean for us? Let's create a new folder `wrappers` and a file called `MainContract.ts` within it. This file will implement and export a wrapper around our contract.
```
mkdir wrappers && cd wrappers && touch MainContract.ts
```
> Make sure you run this command sequence from the root of your project

Open `MainContract.ts` for editing. Let's import a `Contract` interface from `ton-core` library, then define and export a class that will implement `Contract`.
```
import { Contract } from 'ton-core';

export class MainContract implements Contract {
    
}
```

If you take a look into contents of the `Contract` interface - you will see that it expects `address`,`init` and `abi` params. We will only use `address` and `init` for our purposes. To use them, we define a constructor for our `MainContract` class.
> If you have no idea what we are talking about here with classes and constructors - it might be a good idea for you to read more about Object Oriented Programming. FunC doesn't require this, but for better testing with TypeScript you should no these basic concepts.

```
import { Address, Cell, Contract } from "ton-core";

export class MainContract implements Contract {
  constructor(
    readonly address: Address,
    readonly init?: { code: Cell; data: Cell }
  ) {}
}
```

The `init` property is very interesting. It has an initial state of our contract. Code is obviously the code of the contract. Data is a nit more interesting. Remember we were talking about the persistent c4 storage of our contract? With this **data** Cell, we can define what is going to be in this storage once our contract first executes.  Both of this values have `Cell` type as they're store in a memory of TVM during the contract's lifecycle. Same `code` and `data` cells are also used to calculate the future address of our contract.

Note that we also imported classes `Address` and `Cell` from `ton-core` library. 

> In TON the addresses of contracts are deterministic. We can know them before even deploying our contract. We will see how this is done in a few steps.

Now let's define a static method for our `MainContract` class - `createFromConfig`. We are not going to use any config parameters for now, but in future we assume that we will need input data in order to create an instance of a contract.

```
import { Address, beginCell, Cell, Contract, contractAddress } from "ton-core";

export class MainContract implements Contract {
  constructor(
    readonly address: Address,
    readonly init?: { code: Cell; data: Cell }
  ) {}

  static createFromConfig(config: any, code: Cell, workchain = 0) {
    const data = beginCell().endCell();
    const init = { code, data };
    const address = contractAddress(workchain, init);

    return new MainContract(address, init);
  }
}
```

Our `createFromConfig` method is accepting a `config` param (ignore for now), `code` which is a Cell with a compiled code of our contract and a `workchain` which defines a workchain of TON that the contract is ment to be placed on. Currently there is only one workchain on ton - 0.

To better understand this code let's start from the results we return. We are creating a new instance of our class `MainContract` and as defined in it's constructore, we have to pass it contract future **address** (that we can calculate as noted before) and the **init** state of the contract.

The address we calculate by a function that we import from `ton-core` library. We pass `workchain` and `init` state params in order to get it.

The `init` state is simply and object with `code` and `data` properties. Where the `code` is passed is passed into our method and `data` is just an empty Cell for now. We will learn how to transform `config` data into Cell format, so we could use `config` data in `init` state of our contract.

To sum up, the `createFromConfig` is accepting a `config` with data that we will in future store in contract's persistent storage and `code` of the contract. In return, we get an instance of a contract that we can easily interact with help of `sandbox`.

Let's get back to our `tests/main.spec.ts` and do following steps:
- import our wrapper
- import the contract's compiled code as hex representation
- transform it into Cell
- use sandbox's `openContract` along with our new wrapper, to get an instance of contract that we can finally interact with

```
import { Cell } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { Blockchain } from "@ton-community/sandbox";
import { MainContract } from "../wrappers/MainContract";

describe("main.fc contract tests", () => {
  it("our first test", async () => {
    const blockchain = await Blockchain.create();
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    const myContract = blockchain.openContract(
      await MainContract.createFromConfig({}, codeCell)
    );
  });
});

```

At this point, we have an instance of a smartcontract that we can interact with in many ways similar to a real contract, in order to test the expected behaviours.

### Interacting with contract

**ton-core** library provides us with another great construction called **Address**. As you know, every entity on TON blockchain has an address. In real life, if you want to send a message from one contract (ex. wallet contract) to another - you know both contract's addresses.

While writing tests we emulate a contract and when we interact with - the address of our emulated contract is known. However, we will need a wallet that we will use to deploy our contract and another one to interact with our contract.

In `sandbox` this is done in a way that we are calling a `treasure` method of a blockchain instance and provide it with a seed phrase: 
`const senderWallet = await blockchain.treasury("sender");`

### Deploying a contract to an emulated blockchain

???

### Emulating an internal message

Let's get back to writing our test. Since we have an instance of our contract, let's send an internal message to it, so our sender address will be saved in c4 storage. We remember that to interact with our contract instance we need to use the wrapper. Let's get back to our file **wrappers/MainContract.ts** and create a new method in our wrapper called **sendInternalMessage**.

It would look like this initially:

```
async sendInternalMessage(
    provider: ContractProvider,
    sender: Sender,
    value: bigint
  ){
    
}
```

Our new method is receiving param of type **ContractProvider**, the sender of the message of type **Sender** and the value of the message **value**. 

Normally we don't have to worry about passing the **ContractProvider** when using this method, it will be passed under the hood as part of the contract instance built in functinality. However, don't forget to import those types from the **ton-core** library.

Let's implement a logic of sending internal message inside of our new method. It would look like this:

```
async sendInternalMessage(
    provider: ContractProvider,
    sender: Sender,
    value: bigint
  ) {
    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: beginCell().endCell(),
    });
  }
```

You can see that we are using the **provider** and call it's method called **internal**. We pass the **sender** as a first parameter, then we form an object with arguments that includes:
- **value** of message (the amount of TONs in format of nano)
- **sendMode** - we use a SendMode enum provided by `ton-core`, you can read more about how the modes work under the hood in [documentation page](https://ton.org/docs/develop/smart-contracts/guidelines/processing) dedicated to this
- **body** that is supposed to be a cell with body of message, but we keep it an empty cell for now

Please don't forget to import all the new types and entities we are using from the `ton-core` library.

So our final code of **wrappers/main** is looking like this:

```
import { Address, beginCell, Cell, Contract, contractAddress, ContractProvider, Sender, SendMode } from "ton-core";

export class MainContract implements Contract {
  constructor(
    readonly address: Address,
    readonly init?: { code: Cell; data: Cell }
  ) {}

  static createFromConfig(config: any, code: Cell, workchain = 0) {
    const data = beginCell().endCell();
    const init = { code, data };
    const address = contractAddress(workchain, init);

    return new MainContract(address, init);
  }

  async sendInternalMessage(
    provider: ContractProvider,
    sender: Sender,
    value: bigint
  ) {
    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: beginCell().endCell(),
    });
  }
}

```

Calling this method in our **tests/main.spec.ts** would look like this:

```
 
const senderWallet = await blockchain.treasury("sender");
myContract.sendInternalMessage(senderWallet.getSender(), toNano("0.05"));
```

Note how we are using **toNano** helper function imported from `ton-core` library to transform a string value to nano bignum format.


### Invoking a getter method

We will need to create one more method on our contract's wrapper, namely - **getData** method that will be running a getter method of our contract and returing the results we have in c4 storage.

This is how our getter method will look like:

```
async getData(provider: ContractProvider) {
    const { stack } = await provider.get("get_the_latest_sender", []);
    return {
      recent_sender: stack.readAddress(),
    };
}
```

Just like in our case of sending an internal message - we are using provider and it's methods. In this case we are using the **get** method. Then we are reading the address from received **stack** and return it as a result. 

### Writing tests

That's it. We have done all the preparation work and now we will write the actual test logic. Here is how our test scenario is going to look like:

1. We send and internal message
2. We ensure the send was successfull
3. We call a getter method of the contract and ensure the call was successfull.
4. We compare the results received from getter to the **from** from address we've set in the original internal message.

Seems quite simple and doable! Let's go for it.

Sandbox team equiped us with one more amasing test util. We can install additional **@ton-community/test-utils** package by running
`yarn add @ton-community/test-utils -D`

This will allow us to use **.toHaveTransaction** for jest matcher to add additional helpers for ease of testing. We will also need to import that package in our **tests/main.spec.ts** after install.

Let's see how our testing code based on the described above scenario looks like.

```
import { Cell, toNano } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { Blockchain } from "@ton-community/sandbox";
import { MainContract } from "../wrappers/MainContract";
import "@ton-community/test-utils";

describe("main.fc contract tests", () => {
  it("should get the proper most recent sender address", async () => {
    const blockchain = await Blockchain.create();
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    const myContract = blockchain.openContract(
      await MainContract.createFromConfig({}, codeCell)
    );

    const senderWallet = await blockchain.treasury("sender");

    const sentMessageResult = await myContract.sendInternalMessage(
      senderWallet.getSender(),
      toNano("0.05")
    );

    expect(sentMessageResult.transactions).toHaveTransaction({
      from: senderWallet.address,
      to: myContract.address,
      success: true,
    });

    const data = await myContract.getData();

    expect(data.recent_sender.toString()).toBe(senderWallet.address.toString());
  });
});

```

We're using Jest functionality to make sure that:

- the internal message send was successfull
- the getter method was succesfull
- the getter method return some result
- the address returned from the getter method is equal to the one we used in our message setting the **from** address

We've also renamed 'our first test' to 'should get the proper most recent sender address', we want to always keep our tests' names readable.

### Running tests

Voila! We are ready to run our tests. Simply run our command `yarn test` in terminal and if you've done everything with me, you will receive similar result:

```
 PASS  tests/main.spec.ts
  main.fc contract tests
    ✓ should get the proper most recent sender address (444 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        4.13 s, estimated 5 s
```

One last thing we want to do is to make sure that every time we are running tests - we also run our compiler script. This is usefull for effeciency of our work. Most of the developement work is writing func code and then running tests. Let's streamline this:

Update your **package.json** file to look like this:

```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "test": "yarn compile && yarn jest"
  }
}
```

In the next lesson we are going to build our deploy pipeline and learn about how to test the real deploy contract on-chain. See you there!

[Proceed to Lesson 5 >](https://github.com/markokhman/func-course/blob/main/Chapter%203/Lesson%205.md)

[< Go to previous lesson](https://github.com/markokhman/func-course/blob/main/Chapter%203/Lesson%203.md)
