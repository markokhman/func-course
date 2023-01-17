# Chapter 4. Lesson 2.

As I've mentioned before, writing tests is going to be a very solid part of your time programming FunC contracts. Let's recap our tests created in previous lesson:

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

It's worth mentioning, that if we run `yarn test` now, it is not going to work, because our code changed significantly. What we need to do in order to fix that, is following:

1. We need to provide proper initial state data, so our c4 storage has some data prefilled. We are going to provide 0 as the counter value and zeroAddress as the most resent sender's address.
2. We need to pass a specific op command inside of the body of the message we send to the contract.

Let's deal with the initial data. When we first wrote our contract tests, we kept it as an empty cell:

```
 const initDataCell = beginCell().endCell();
```

Let's create a variable **counter_initial_value** and place data in the order that our contract expects to see when reading c4 storage:

```
 const counter_initial_value = 0;
 const initDataCell = beginCell().storeUint(counter_initial_value, 32).storeAddress(zeroAddress).endCell();
```

> Keep in mind that the **zeroAddress** needs to be imported from the **./helpers.ts** file.

The message body also needs to be updated from:

```
const messageBody = new Cell();
```

to

```
const messageBody = begin;
```

Since we've changed the data our getter method returns, we would need to adjust it's test. What we had previously:

```
expect(call.result.length).toBe(1);

if (call.result.length === 0) {
  throw new Error("No results returned from getter method");
}

const most_recent_sender = (call.result[0] as Slice).readAddress();
```

Now, we know that the amount of returned values is 2, let's adjust test accordingly. We also can read the counter value and create a test for what we would expect from it - to be incremented by 1 from the **inital_counter_value**:

```
expect(call.result.length).toBe(2);

if (call.result.length === 0) {
  throw new Error("No results returned from getter method");
}

const counter_value = call.result[0] as BN;
const most_recent_sender = (call.result[1] as Slice).readAddress();

expect(counter_value.toNumber()).toBe(counter_initial_value + 1);

expect(most_recent_sender?.toFriendly()).toBe(
  randomAddress("notowner").toFriendly()
);
```

Great, our final code is looking like this:

```
import { Cell, beginCell, CommonMessageInfo, InternalMessage, CellMessage, Slice } from "ton";
import { hex } from "../build/main.compiled.json";
import { SmartContract } from "ton-contract-executor";
import { randomAddress, zeroAddress } from "./helpers";
import BN from "bn.js";

describe("main.fc contract tests", () => {
  it("should successfully increase counter in contract and get the proper most recent sender address", async () => {
    const counter_initial_value = 0;

    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    const initDataCell = beginCell()
      .storeUint(counter_initial_value, 32)
      .storeAddress(zeroAddress)
      .endCell();
    const contract = await SmartContract.fromCell(codeCell, initDataCell);

    const messageBody = beginCell().storeUint(1, 32).endCell();

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

    expect(call.result.length).toBe(2);

    if (call.result.length === 0) {
      throw new Error("No results returned from getter method");
    }

    const counter_value = call.result[0] as BN;
    const most_recent_sender = (call.result[1] as Slice).readAddress();

    expect(counter_value.toNumber()).toBe(counter_initial_value + 1);

    expect(most_recent_sender?.toFriendly()).toBe(
      randomAddress("notowner").toFriendly()
    );
  });
});

```

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
