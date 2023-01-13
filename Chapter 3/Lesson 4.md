# Chapter 3. Lesson 4.

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
import { Cell } from "ton-core";
import { hex } from "../build/main.compiled.json";

describe("main.fc contract tests", () => {

  it("our first test", async () => {
 	const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    	const contract = await SmartContract.fromCell(code, new Cell());
  });

});

```

To be continued... Need to figure out why InternalMessage is not part of the **ton** library anymore.
