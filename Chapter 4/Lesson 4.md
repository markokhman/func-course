# Chapter 4. Lesson 4.
###### tags: `Chapter 4`

In this lesson we are going to write quite a few interesting tests. 

We are going to first update the initialization process for our previously written contract, because we have introduced a new variable, that is supposed to be stored in c4 storage.

If you try to run our test now - `yarn test` and you can see that the test is going to fail. You will also see trace of transactions that happened to our contract and one of them will have the error code 9. [Have a look at the exit codes documentation.](https://ton.org/docs/learn/tvm-instructions/tvm-exit-codes)

> 9 - Cell underflow. Read from slice primitive tried to read more bits or references than there are.

### Updating our contract wrapper

First of all let's update our contract wrapper, so it would be aware of this third parameter we have in the c4 storage. Namely we will update **MainContractConfig** and **mainContractConfigToCell** functions

```
// ...library imports

export type MainContractConfig = {
  number: number;
  address: Address;
  owner_address: Address;
};

export function mainContractConfigToCell(config: MainContractConfig): Cell {
  return beginCell()
    .storeUint(config.number, 32)
    .storeAddress(config.address)
    .storeAddress(config.owner_address)
    .endCell();
}

// ...contact wrapper class
```

One more thing to update in the wrapper file - **getData** method, so it would also return the `owner_address` and we will also create new method **getBalance**, to test the balance was increased after deposit:

```
async getData(provider: ContractProvider) {
    const { stack } = await provider.get("get_contract_storage_data", []);
    return {
      number: stack.readNumber(),
      recent_sender: stack.readAddress(),
      owner_address: stack.readAddress(),
    };
  }
  
async getBalance(provider: ContractProvider) {
    const { stack } = await provider.get("balance", []);
    return {
      number: stack.readNumber(),
    };
}
```

Let's introduce a new treasury that is going to be actually the owner of contract and only this owner will be able to withdraw funds later on. These updates are made in **tests/main.spec.ts**:
```
// ...library imports

describe("main.fc contract tests", () => {
  it("should get the proper most recent sender address", async () => {
    const blockchain = await Blockchain.create();
    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    const initAddress = await blockchain.treasury("initAddress");
    const ownerAddress = await blockchain.treasury("ownerAddress");
 
 // ...rest of testing code
 
```

Now we are good. `yarn test` successfully passes our previously written test.

### Tests for depositing funds

Let's add a few more tests in our **main.spec.ts** file. 

Since we have to re-initiate a contract before each new test, we will also bring the contract initiation into Jest's **beforeEach** function:


```
// ... library imports
describe("main.fc contract tests", () => {
  let blockchain: Blockchain;
  let myContract: SandboxContract<MainContract>;
  let initWallet: SandboxContract<TreasuryContract>;
  let ownerWallet: SandboxContract<TreasuryContract>;

  beforeEach(async () => {
    blockchain = await Blockchain.create();
    initWallet = await blockchain.treasury("initWallet");
    ownerWallet = await blockchain.treasury("ownerWallet");

    const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];

    myContract = blockchain.openContract(
      await MainContract.createFromConfig(
        {
          number: 0,
          address: initWallet.address,
          owner_address: ownerWallet.address,
        },
        codeCell
      )
    );
  });

  it("should get the proper most recent sender address", async () => {
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
  it("successfully deposits funds", () => {
    // test logic is coming
  });
  it("should return deposit funds as no command is sent", () => {
    // test logic is coming
  });
  it("successfully withdraws funds on behalf of owner", () => {
    // test logic is coming
  });
  it("fails to withdraw funds on behalf of non-owner", () => {
    // test logic is coming
  });
  it("fails to withdraw funds because lack of balance", () => {
    // test logic is coming
  });
});
```
> We simply moved part of the first test's lines into the **beforeEach**, the rest of lines should stay inside the first test **it**


Let's write a test for our deposit. We are going to send a message with **op** == 2 and then check the balance. As always - let's first create a wrapper method, we will call it **sendDeposit**:

```
async sendDeposit(provider: ContractProvider, sender: Sender, value: bigint) {
    const msg_body = beginCell()
      .storeUint(2, 32) // OP code
      .endCell();

    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: msg_body,
    });
  }
```

We will not go into details of this method, as it is extremely simple - we just pass an op code inside of the message body.

Here is how we use this method to run deposit tests.

```
 it("successfully deposits funds", async () => {
    const senderWallet = await blockchain.treasury("sender");

    const depositMessageResult = await myContract.sendDeposit(
      senderWallet.getSender(),
      toNano("5")
    );

    expect(depositMessageResult.transactions).toHaveTransaction({
      from: senderWallet.address,
      to: myContract.address,
      success: true,
    });

    const balanceRequest = await myContract.getBalance();

    expect(balanceRequest.number).toBeGreaterThan(toNano("4.99"));
  });
```

Note how we are checking that the balance of contract is more then 4.99 TONs as we know that some funds will be lost on comissions.

Let's write one more test for deposit, but this time it will be false positive. We are going to send funds without deposit op code and expect a return transaction, as this op code is unkonwn.

Just like always - we have one more wrapper:
```
async sendNoCodeDeposit(
    provider: ContractProvider,
    sender: Sender,
    value: bigint
  ) {
    const msg_body = beginCell().endCell();

    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: msg_body,
    });
  }
```

and here is the test code:

```
it("should return funds as no command is sent", async () => {
    const senderWallet = await blockchain.treasury("sender");

    const depositMessageResult = await myContract.sendNoCodeDeposit(
      senderWallet.getSender(),
      toNano("5")
    );

    expect(depositMessageResult.transactions).toHaveTransaction({
      from: myContract.address,
      to: senderWallet.address,
      success: true,
    });

    const balanceRequest = await myContract.getBalance();

    expect(balanceRequest.number).toBe(0);
});
```

### Funds withdrawal tests

Let's first create a new wrapper for withdrawal of funds:
```
async sendWithdrawalRequest(
    provider: ContractProvider,
    sender: Sender,
    value: bigint,
    amount: bigint
  ) {
    const msg_body = beginCell()
      .storeUint(3, 32) // OP code
      .storeCoins(amount)
      .endCell();

    await provider.internal(sender, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: msg_body,
    });
  }
```

We put a proper `msg_body` and the desired `amount` to be withdrawn.

We are going to have 3 different tests. Let's have a look:

```
it("successfully withdraws funds on behalf of owner", async () => {
    const senderWallet = await blockchain.treasury("sender");

    await myContract.sendDeposit(senderWallet.getSender(), toNano("5"));

    const withdrawalRequestResult = await myContract.sendWithdrawalRequest(
      ownerWallet.getSender(),
      toNano("0.05"),
      toNano("1")
    );

    expect(withdrawalRequestResult.transactions).toHaveTransaction({
      from: myContract.address,
      to: ownerWallet.address,
      success: true,
      value: toNano(1),
    });
  });
```

To successfully withdraw funds - we first deposit them properly. Then we call `sendWithdrawalRequest` specifying that we want to withdraw 1 TON. Note that 0.05 TONs are specified simply for message value to pay the fees.

```
it("fails to withdraw funds on behalf of not-owner", async () => {
    const senderWallet = await blockchain.treasury("sender");

    await myContract.sendDeposit(senderWallet.getSender(), toNano("5"));

    const withdrawalRequestResult = await myContract.sendWithdrawalRequest(
      senderWallet.getSender(),
      toNano("0.5"),
      toNano("1")
    );

    expect(withdrawalRequestResult.transactions).toHaveTransaction({
      from: senderWallet.address,
      to: myContract.address,
      success: false,
      exitCode: 103,
    });
  });
```

In this test we are sending withdrawal request on behalf of senderWallet, rather then ownerWallet (those differe because of seed phrase we provide once we initialize them).

Also note how we are waiting for a transaction to faile with **success: false** and specific reason of failure **exitCode: 103**. To make sure we do it right - have a look at our contract code, we have such a line there:
```
throw_unless(103, equal_slice_bits(sender_address, owner_address));
```

Let's proceed to the last test - withdrawal fail because lack of balance:

```
it("fails to withdraw funds because lack of balance", async () => {
    const withdrawalRequestResult = await myContract.sendWithdrawalRequest(
      ownerWallet.getSender(),
      toNano("0.5"),
      toNano("1")
    );

    expect(withdrawalRequestResult.transactions).toHaveTransaction({
      from: ownerWallet.address,
      to: myContract.address,
      success: false,
      exitCode: 104,
    });
  });
```

In this test we removed the deposit logic, so the contract has 0 funds in the beginning. This is why we expect it to fail with **exitCode: 104**. Let's again reference to the line in our contract that is throwing this error:
```
throw_unless(104, balance >= withdraw_amount);
``` 

That's it :)

You've made it bro! We are very likely going to expand this Chapter 4 with more lessons that will cover more and more topics of the most widely used logic pieces.

All the code of this chapter is available in [this repository](https://github.com/markokhman/func-course-chapter-4-code).

After a few chapters we are going to learn how to build web clients for interacting with your contracts.


