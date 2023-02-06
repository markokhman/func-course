# Chapter 3. Lesson 4.

**TODO: This part must be adjusted, as the versioning of ton, ton-core and ton-contract-executor is not clear**

We've written our first FunC contract, we know that it succesfully compiles. What's next?

In this lesson we will learn how to make sure our contract's code is actuall working as expected. Let me remind you our expectations to the code:

Our contract is supposed to save a sender address every time it receives a message and it should return the latest sender's address once we call the getter method.

Great, but how to we ensure that it is working? Well, we could deploy it to the blockchain (which we will eventually do very soon, in the next lesson), but TON has some tools, that allow us simulate certain behaviours locally.

This is done with help of `ton-contract-executor`. This library allows you to run TON Virtual Machine locally and execute contract. Because we are having our typescript "laboratory", we can create a sequence of tests with help of another library - `jest`. This way, we have a test suite that simulates all the important behaviours, with different input data and checks the results. This is the best approach to write & debug & fully test your contracts before launching them to the network.

### Preparing our test suite

First of all, assuming you are currently in the root of our project, let's install `ton-contract-executor` and `jest`:

```
yarn add ton-contract-executor jest @types/jest --dev
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

In case you've never written TypeScript tests, you should definitely check try this approach. While programmin TON Smartcontracts your are going to spend more then half of your programmin time on writing tests. Remember our Space satellite example? Same here, we simulate every important case before we ever deploy the contract even to testnet.

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

In order to write our first test, we need to understand how we can create a TypeScript instance of our compiled contract with help of `ton-contract-executor`.

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

Now let's take a look at **ton-contract-executor's** [quick start guide](https://github.com/ton-community/ton-contract-executor). We see that we can import a class **SmartContract** and it has a method **.fromCell**, that is expecting to receive two Cells, namely **code** Cell and **data** Cell.

We already know what is **codeCell** and how to construct it. What is this **data** Cell? Remember we were talking about the persistent c4 storage of our contract? With this **data** Cell, we can define what is going to be in this storage once our contract first executes. Awesome, isn't it?

We don't need this functionality for now, so we will pass just an empty Cell, but I assure you that we will have lot's of use from this feature.

So, let's create an instance of our contract:

```
import { Cell, beginCell } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { SmartContract } from "ton-contract-executor";

describe("main.fc contract tests", () => {

  it("our first test", async () => {
 	const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    const initDataCell = beginCell().endCell();
    const contract = await SmartContract.fromCell(codeCell, initDataCell);
  });
});

```

At this point, we have an instance of a smartcontract that we can interact with in many ways similar to a real contract, in order to test the expected behaviours.

### Working with Address type while writing tests

**ton** library provides us with another great construction called **Address**. As you know, every entity on TON blockchain has an address. In real life, if you want to send a message from one contract (ex. wallet contract) to another - you know both contract's addresses.

While writing tests we emulate a contract and when we interact with - there is no need of an adress, as our emulated contract is known. However, then we will need to specify an address, while constructing messages that we send to our contract while testing it. We are going to use a so called "zero address". Zero address has a same format as real address, but is filled with 0 instead of actual address. This is simply a placeholder:

`const zeroAddress = new Address(0, Buffer.alloc(32, 0));`

But sometimes we will need to generate different addresses. In our case it would be a scenario:

1. We compose an internal message and specifying a certain **from** address
2. We send this message to our contract
3. We call **get_the_latest_sender** getter method

Expected behaviour is that the address we get from getter method will be equal to the one we set as **from** address in Step 1. So we need some utility that would enable us quickly generate addresses that we are going to use in our testing functionality.

We are going to create a file **tests/helpers.ts** and locate those utilities in it:

```
import { Address } from "ton";
import Prando from "prando";

export const zeroAddress = new Address(0, Buffer.alloc(32, 0));

export function randomAddress(seed: string, workchain?: number) {
  const random = new Prando(seed);
  const hash = Buffer.alloc(32);
  for (let i = 0; i < hash.length; i++) {
    hash[i] = random.nextInt(0, 255);
  }
  return new Address(workchain ?? 0, hash);
}
```

You might have noticed that we're using a library **Prando**. Prando is a deterministic pseudo-random number generator. It can be used to create a series of random numbers that can later be re-created given the same seed.

You would need to install it with a command:

```
yarn add prando --dev
```

### Emulating of internal message

Let's get back to writing our test. Since we have an instance of our contract, let's send an internal message to it, so our sender address will be saved in c4 storage:

```
import { Cell, beginCell, InternalMessage, CommonMessageInfo, CellMessage } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { SmartContract } from "ton-contract-executor";
import { randomAddress, zeroAddress } from "./helpers";

describe("main.fc contract tests", () => {

  it("our first test", async () => {
 	const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    	const initDataCell = beginCell().endCell();
    	const contract = await SmartContract.fromCell(codeCell, initDataCell);
  });

  const message = new InternalMessage({
      to: zeroAddress,
      from: randomAddress("notowner"),
      value: 0,
      bounce: false,
      body: new CommonMessageInfo({ body: new CellMessage(messageBody) }),
    });

  const result = await contract.sendInternalMessage(message);

});
```

As you can see, we've imported 3 new types from **ton** library - InternalMessage, CommonMessageInfo and CellMessage. All 3 are helping use to construct an internal message.

One more thing we need to do while preparing for actual test is to call the **get_the_latest_sender** getter. This is done with a single command:

```
const call = await contract.invokeGetMethod("get_the_latest_sender", []);
```

### Writing tests

That's it. We have done all the preparation work and now we will write the actual test logic. Here is how our test scenario is going to look like:

1. We send and internal message with a specified **from** address
2. We ensure the send was successfull
3. We call a getter method of the contract and ensure the call was successfull.
4. We compare the results received from getter to the **from** from address we've set in the original internal message.

Seems quite simple and doable! Let's go for it.

```
import { Cell, beginCell, CommonMessageInfo, InternalMessage, CellMessage, Slice } from "ton";
import { hex } from "../build/main.compiled.json";
import { SmartContract } from "ton-contract-executor";
import { randomAddress, zeroAddress } from "./helpers";

describe("main.fc contract tests", () => {
  it("should get the proper most recent sender address", async () => {
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    const initDataCell = beginCell().endCell();
    const contract = await SmartContract.fromCell(codeCell, initDataCell);

    const messageBody = new Cell();

    const message = new InternalMessage({
      to: zeroAddress,
      from: randomAddress("notowner"),
      value: 0,
      bounce: false,
      body: new CommonMessageInfo({ body: new CellMessage(messageBody) }),
    });

    const result = await contract.sendInternalMessage(message);

    expect(result.type).toBe("success");

    const call = await contract.invokeGetMethod("get_the_latest_sender", []);

    expect(call.type).toBe("success");

    if (call.type !== "success") {
      throw new Error("Getter method call failed");
    }

    expect(call.result.length).toBe(1);

    if (call.result.length === 0) {
      throw new Error("No results returned from getter method");
    }

    const most_recent_sender = (call.result[0] as Slice).readAddress();

    expect(most_recent_sender?.toFriendly()).toBe(
      randomAddress("notowner").toFriendly()
    );
  });
});
```

You can see that we've imported one more thing from **ton** library - Slice. As you remember, addresses have a slice type in TON. The result we get from the getter method is a slice, but in order to compare the address it contains - we need to treat the result as a slice and read the address from it.

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
