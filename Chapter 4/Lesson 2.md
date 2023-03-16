# Chapter 4. Lesson 2.
###### tags: `Chapter 4`

As I've mentioned before, writing tests is going to be a very solid part of your time programming FunC contracts. Let's recap our tests created in previous lesson:

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

It's worth mentioning, that if we run `yarn test` now, it is not going to work, because our code changed significantly. What we need to do in order to fix that, is following:
1. We need to provide proper initial state data, so our c4 storage has some data prefilled. We are going to provide 0 as the counter value and zeroAddress as the most resent sender's address (according to a new logic of our contract).
2. We need to pass a specific op command inside of the body of the message we send to the contract.

Let's deal with the initial data. As you remember in our testing suite (Sandbox) we are using a **createFromConfig** method to initialise our contract and we are setting the initial state data for our contract as simple empty cell in following way:
```
static createFromConfig(config: any, code: Cell, workchain = 0) {
    const data = beginCell().endCell();
    const init = { code, data };
    const address = contractAddress(workchain, init);

    return new MainContract(address, init);
}
```

Then the **createFromConfig** is called in our **main.spec.ts**:
```
const myContract = blockchain.openContract(
  await MainContract.createFromConfig({}, codeCell)
);
```

So far we were passing an empty object as a config, but let's now allow passing values that we are actually going to need to initialise our contract's proper data.

Let's define what our config is supposed to look like. Right after imports in our **wrappers/MainContract.ts** we are going to define a new type called **MainContractConfig**. 

```
// ... library imports

export type MainContractConfig = {
  number: number;
  address: Address;
};
```

Another cool helper function we are going to implement in the same wrapper file is going to be **mainContractConfigToCell**. It is going to receive a **MainContractConfig** type of object, properly pack those parameters into a cell and return it. We need this because as you remember, the c4 storage is a memory cell:

```
// ... library imports

// ... MainContractConfig type

export function mainContractConfigToCell(config: MainContractConfig): Cell {
  return beginCell().storeUint(config.number, 32).storeAddress(config.address).endCell();
}
```


Let's now update the **createFromConfig** in the same file, but inside of the **MainContract** class. It now has to use the **mainContractConfigToCell** function to form the initial state data cell:

```
// ... library imports
// ... MainContractConfig type
// ... mainContractConfigToCell function

export class MainContract implements Contract {
  constructor(
    readonly address: Address,
    readonly init?: { code: Cell; data: Cell }
  ) {}

  static createFromConfig(
    config: MainContractConfig,
    code: Cell,
    workchain = 0
  ) {
    const data = mainContractConfigToCell(config);
    const init = { code, data };
    const address = contractAddress(workchain, init);

    return new MainContract(address, init);
  }
  
  // ... rest of the methods
}
```

Now, as our **createFromConfig** method is ready, let's update how we are using it inside of our **tests/main.spec.ts** file:

```
// ... library imports

describe("main.fc contract tests", () => {
  it("should get the proper most recent sender address", async () => {
    const blockchain = await Blockchain.create();
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    const initAddress = await blockchain.treasury("initAddress");

    const myContract = blockchain.openContract(
      await MainContract.createFromConfig(
        {
          number: 0,
          address: initAddress.address,
        },
        codeCell
      )
    );
    
    // ... the rest of old tests logic
    
}
```

Note how we've created an **initAddress** wallet (treasury) from a seed "initAddress" to pass it's address into the **createFromConfig** method of our **MainContract**.

At this point we are successfully initialising our contract! Congrats :)

As we are now also going to send a certain data inside of the message body - let's update the method **sendInternalMessage** in our **wrappers/MainContract.ts**.

We are going to rename it to **sendIncrement** and it will now accept one more value - **increment_by**. This is going to be a number for the current contract's storage value to be increment by.

We also keep in mind that this is the same body that our contract is parsing in order to get the operation code. So, let's not forget to initially store a 32-bit integer (1 for incrementing, as you can see in our contract's code) and then - the 32-bit integer for the incrementing value:

```
async sendIncrement(
    provider: ContractProvider,
    sender: Sender,
    value: bigint,
    increment_by: number
  ) {
    
    const msg_body = beginCell()
      .storeUint(1, 32) // OP code
      .storeUint(increment_by, 32) // increment_by value
      .endCell();

    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: msg_body,
    });
  }
```

There is one more method we need to update in our wrapper - **getData**. As you remember, we've renamed the getter method and also it returs two values now. Let's adjust our wrapper method:

```
async getData(provider: ContractProvider) {
    const { stack } = await provider.get("get_contract_storage_data", []);
    return {
      number: stack.readNumber(),
      recent_sender: stack.readAddress(),
    };
  }
```

Note, how we read the resulting stack of the getter method. We us **readNumber()** to read the integer of the current value.

Now we are all set to return to our **tests/main.spec.ts** and proceed with completing our test. Here is the updated code of our test:

```
// ... library imports 

describe("main.fc contract tests", () => {
  it("should get the proper most recent sender address", async () => {
    const blockchain = await Blockchain.create();
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    const initAddress = await blockchain.treasury("initAddress");

    const myContract = blockchain.openContract(
      await MainContract.createFromConfig(
        {
          number: 0,
          address: initAddress.address,
        },
        codeCell
      )
    );

    const senderWallet = await blockchain.treasury("sender");

    const sentMessageResult = await myContract.sendIncrement(
      senderWallet.getSender(),
      toNano("0.05"),
      1
    );

    expect(sentMessageResult.transactions).toHaveTransaction({
      from: senderWallet.address,
      to: myContract.address,
      success: true,
    });

    const data = await myContract.getData();

    expect(data.recent_sender.toString()).toBe(senderWallet.address.toString());
    expect(data.number).toEqual(1);
  });
});

```

What we've updated here?

1. We now pass an number - 1 - to the **sendIncrement** method, so our resulting value must be incremented by 1.
2. We've set expectations for the **recent_sender** and **number** values.

We are now able to run our test with `yarn test` command. If you've done everything right - you will see following in the terminal:
```
  PASS  tests/main.spec.ts
  main.fc contract tests
    ✓ should successfully increase counter in contract and get the proper most recent sender address (452 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        4.339 s, estimated 5 s
```

Voila, our counter contract is now properly tested. Looking forward for some more logic in the next lessons!

