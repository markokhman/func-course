# Chapter 4. Lesson 4.

In this lesson we are going to write quite a few interesting tests.

We are going to first update our previously written contract, because we have introduced a new variable, that is supposed to be stored in c4 storage.

If you try to run our test now - `yarn test` and you console log the resulting code of our contract's **sendInternalMessage** method - you will see that the error code is 9. [Have a look at the exit codes documentation.](https://ton.org/docs/learn/tvm-instructions/tvm-exit-codes)

> 9 - Cell underflow. Read from slice primitive tried to read more bits or references than there are.

Let's introduce a new variable with a random addres from seed "owner" and place it into the init data cell:

```
 const OWNER_ADDRESS = randomAddress("owner");

 //other code

 const initDataCell = beginCell()
      .storeUint(counter_initial_value, 32)
      .storeAddress(zeroAddress)
      .storeAddress(OWNER_ADDRESS)
      .endCell();
```

Another important thing we need to update is the amount of results we are awaiting from the getter method. Since we introduced the **owner_address** into the storage, it should be 3.

```
expect(call.result.length).toBe(3);
```

Now we are good. `yarn test` successfully passes our test.

### Tests for deposit and withdrawal

Let's add three more tests in our **main.spec.ts** file. Since we have to re-initiate a contract before each new test, we will also bring the contract initiation into Jest's **beforeEach** function:

```
describe("main.fc contract tests", () => {
  let contract: SmartContract;
  const counter_initial_value = 0;

  beforeEach(async () => {
    const OWNER_ADDRESS = randomAddress("owner");

    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
    const initDataCell = beginCell()
      .storeUint(counter_initial_value, 32)
      .storeAddress(zeroAddress)
      .storeAddress(OWNER_ADDRESS)
      .endCell();
    contract = await SmartContract.fromCell(codeCell, initDataCell);
  });
  it("should successfully increase counter in contract and get the proper most recent sender address", async () => {
    // this test is working properly
  });
  it("successfully deposits funds", () => {
    // test logic is coming
  });
  it("successfully withdraws funds on behalf of owner", () => {
    // test logic is coming
  });
  it("fails to withdraw funds on behalf of owner", () => {
    // test logic is coming
  });
});
```

> We simply moved part of the first test's lines into the **beforeEach**, the rest of lines should stay inside the first test **it**

Let's write a test for our deposit. We are going to send a message with **op** == 2 and then check the balance:

```
it("successfully deposits funds", async () => {
    const messageBody = beginCell().storeUint(2, 32).endCell();

    const message = new InternalMessage({
      to: zeroAddress,
      from: randomAddress("notowner"),
      value: toNano(5),
      bounce: false,
      body: new CommonMessageInfo({ body: new CellMessage(messageBody) }),
    });

    const result = await contract.sendInternalMessage(message);

    expect(result.exit_code).toBe(0);

    const call = await contract.invokeGetMethod("get_balance", []);

    expect(call.type).toBe("success");

	// to be continued
});
```

TODO: This lesson needs to be finalized with new **ton, ton-core and tx-emulator**, makes no sense to finish last 2 tests with current versioning.
