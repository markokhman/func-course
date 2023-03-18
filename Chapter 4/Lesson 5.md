## Chapter 4. Lesson 5. Using Blueprint deploy contracts
###### tags: `Chapter 4`
\
As you remember, in the very begining of our FunC programming journey, we've quickly noted that there is a great library/tool called [Blueprint](https://github.com/ton-community/blueprint).

Blueprint is maintained by the TonTech team, officially supported by TON Foundation. We've created our custom scripts from scratch in order to make sure that you completely understand what exactly happens under the hood in projects generated by Blueprint.

It's even recommended to start every of your new projects with following command:
``` 
npm create ton@latest 
```

This command will use Blueprint to generate a new project for you with code for all the phases that we went through in Chapter 3, while covering the contract development lifecycle. 

In this chapter we are not going to slightly optimise our project's setup, so it would work exactly like projects generated with Blueprint. Namely, we are going to slightly update our project in terms of compilation and deployment, so it would use components imported from Blueprint.

### Delegating compilation process to Blueprint

First of all - we will need to update our `package.json` file. From no on, we will only leave there one sript - **test**. We do this, because, as mentioned before, the compilation and deployment will be handled by components imported from Blueprint.

Scripts part of your `package.json` file will now look like this:
```
"scripts": {
    "test": "jest",
}
```

However, if we will run the `yarn test` command now - it will simply run tests on the contract without pre-compiling it. Let's fix that. 

As I've mentioned - we will use comilation suite from Blueprint, so let's install this amazing library:
```
yarn add @ton-community/blueprint --dev
```
In our **tests/main.spec.ts** file we have to make some important updates. This is how it was looking before (we quote here only the beginning part, before actual tests):

```
import { Cell, toNano } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { Blockchain, SandboxContract, TreasuryContract } from "@ton-community/sandbox";
import { MainContract } from "../wrappers/MainContract";
import "@ton-community/test-utils";

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
  
  
  // ... the rest of testing code
 } 
```

We will make 3 updates to it:
1. Import `compile` function from **@ton-community/blueprint** library
2. Implement jest's **beforeAll** hook to compile our code and provide a code cell to the rest of hooks and tests
3. Update the **beforeEach** hook to use the code cell provided by **beforeAll** hook.


This is how our code will transform after those changes are made:

```
// ...other library imports
import { compile } from "@ton-community/blueprint";

describe("main.fc contract tests", () => {
  let blockchain: Blockchain;
  let myContract: SandboxContract<MainContract>;
  let initWallet: SandboxContract<TreasuryContract>;
  let ownerWallet: SandboxContract<TreasuryContract>;
  let codeCell: Cell;

  beforeAll(async () => {
    codeCell = await compile("MainContract");
  });

  beforeEach(async () => {
    blockchain = await Blockchain.create();
    initWallet = await blockchain.treasury("initWallet");
    ownerWallet = await blockchain.treasury("ownerWallet");

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
  
 // ... the rest of testing code 
}
```

> We are doing this changes so you would get used to code that would be generated in case you run `npm create ton@latest`. We are deligating the compilation process to the Blueprint library.

However, we will need to provide Blueprint with one more thing - **compiler config**. To do so, simply create a new file in **wrappers** folder and call it **MainContract.compile.ts**. Here is the contents of this file:
```
import { CompilerConfig } from "@ton-community/blueprint";

export const compile: CompilerConfig = {
  targets: ["contracts/main.fc"],
};
```

At this point you can simply remove the compile scripts file (**scripts/compile.ts**) and run **yarn test** in the root of your project. You will see that our tests are now executing properly! Congratulations!


### Delegating the deployment process to Blueprint

Next thing we need to do is to update our deploy script use Blueprint's superpowers. Quick disclaimer - we are going to use our wrapper **MainContract** and it's method **.createFromConfig**. This way we can easily set the initial state of our contract before deploying it.

Blueprint is bringing amazing features for deployment. It is interactively asking us to choose network (testnet/mainnet) as well as type of wallet that we want to deploy our contract with. This functionality is very handful once you develop and deploy often.

One thing we need to do before updating our deploy script is to create a new method on our wrapper called **sendDeploy**. As you remember, we need to use a wrapper to interact with our contract. This is what our method's code looks like:

```
async sendDeploy(provider: ContractProvider, via: Sender, value: bigint) {
    await provider.internal(via, {
      value,
      sendMode: SendMode.PAY_GAS_SEPARATELY,
      body: beginCell().endCell(),
    });
}
```

As you remember, deploying a contract is as simple as sending it's initial data and code to it's pre-determined address. In Chapter 3 we were composing this manually, so you would understand in depth how it works. Now, for sake of optimisation, we deligate this to our wrapper and Blueprint.


We will completely replace our code with new, so it would comply with Blueprint's functinality.

Here is how our **scripts/deploy.ts** looks like now:

```
import { address, toNano } from "ton-core";
import { MainContract } from "../wrappers/MainContract";
import { compile, NetworkProvider } from "@ton-community/blueprint";

export async function run(provider: NetworkProvider) {
  const myContract = MainContract.createFromConfig(
    {
      number: 0,
      address: address("kQDU69xgU6Mj-iNDHYsWWuNx7yRPQC_bNZNCpq5yVc7LiE7D"),
      owner_address: address(
        "kQDU69xgU6Mj-iNDHYsWWuNx7yRPQC_bNZNCpq5yVc7LiE7D"
      ),
    },
    await compile("MainContract")
  );

  const openedContract = provider.open(myContract);

  openedContract.sendDeploy(provider.sender(), toNano("0.05"));

  await provider.waitForDeploy(myContract.address);
}

```

As you can see - it is much simpler and we are deligating Blueprint so many things that we had to deal with before.

One of coolest things is that now we can use the same **.createFromConfig** method of our wrapper to initiate the contract that we are going to deploy along with the initial state data via **config** object that we pass into it.

Two more things worth of explanations are related to the contents of the config object. 
1. As you remember, our initial data of the contract has to have three params - current counter value (number), the most recent sender (address) and contract's "owner" address - the one who is able to withdraw funds. Because we are going to use this contract on-chain - I've set my real testnet address for both fields. This will enable me to withdraw funds later on.
2. function **address** is helping us to parse a string address into a proper type of Address as our config requests

Core deploy functionality is handled by **provider** of type **NetworkProvider** that is passed by Blueprint into the **run** function when Blueprint is triggering our deploy script.

> All your scripts have to export **run** function if you want them to work with Blueprint

> Let's jump back to **.sendDeploy** method of our wrapper for a second. Have a look at **body** param. You can see that we are sending an empty cell as message body. As of the logic of our contract, if the message body has no operation code set to it - the message will be bounced even though the contract will be successfully deployed. You will see this behaviour in a few moments.

### Deploying contract with help of Blueprint

Let's get to the root of our project and run a command:

```
yarn blueprint run
```

You would expect to see something like this in it after running this command:

```
markokhman$ yarn blueprint run

yarn run v1.22.11

? Choose file to use (Use arrow keys)
❯ deploy 
  onchaintest 
```


> You will see that Blueprint is prompting us via command line to choose the script we want to run. At this moment we still have script **onchaintest.ts**, so it's also offered to us for execution as Blueprint is catching all available scripts from **scripts** folder. We are not going to create any onchain tests at the moment, because in next chapter we are going to create real web interface for interacting with our contract. You can simply remove file **scripts/onchaintest.ts** for now. 


Choose script **deploy**.

You will be prompted to choose network now:

```
markokhman$ yarn blueprint run

yarn run v1.22.11

? Choose file to use (Use arrow keys)
? Choose file to use deploy
? Which network do you want to use? (Use arrow keys)
❯ mainnet 
  testnet 
```

Choose network **testnet**.

```
markokhman$ yarn blueprint run

yarn run v1.22.11

? Choose file to use (Use arrow keys)
? Choose file to use deploy
? Which network do you want to use? 
? Which network do you want to use? testnet
? Which wallet are you using? (Use arrow keys)
❯ TON Connect compatible mobile wallet (example: Tonkeeper) 
  Create a ton:// deep link 
  Tonhub wallet 
  Mnemonic 
```

Now you're prompted to choose which wallet you would like to deploy your contract with. In Chapter 3 we were using only one wallet (TON Whales' Tonhub/Sandbox) and in the most simple manner - creating a deep link.

Blueprint enables you to use any type of wallets with best in class sessions keeping. Namely, it works in a way that you are authorising your local project to request transactions from your wallet and then approve them.

For this tutorial we are going to use TON Whales Sandbox (testnet) and Tonhub (mainnet) again.

Choose Tonhub wallet (keep in mind, on **testnet** version you will need to use **Sandbox app** rather then TonHub app).

You will be asked to scan the QR code and authorize your app to interact with your wallet (request transactions).

```
markokhman$ yarn blueprint run
yarn run v1.22.11
? Choose file to use (Use arrow keys)
? Choose file to use deploy
? Which network do you want to use? (Use arrow keys)
? Which network do you want to use? mainnet
? Which wallet are you using? 
? Which wallet are you using? Tonhub wallet

▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
█ ▄▄▄▄▄ ██▀▀▄▄▀ █▀▀▄█▀███▄  █▀█ ▄▄▄▄▄ █
█ █   █ █▀███ ▀▄█▀▀█▄▀ █▀▀▀▀▄ █ █   █ █
█ █▄▄▄█ █▄▄█▀▀▄▀█▄ ▄▄ █▀██▄▀▄▄█ █▄▄▄█ █
█▄▄▄▄▄▄▄█▄█ ▀▄█ ▀ █ █ █ ▀ █ █▄█▄▄▄▄▄▄▄█
█▄ ▀▀█▄▄▄█▄█▄▀▀▀▀▀ ▀  ▄▄▀█  ▄▄▀▀▀  ████
█▄ ▀ ▀█▄█▄▀▄█▀ ▄▄▄▄▀█▀█▀█ ██▄ ▀ ▀▄▀▄▀▄█
██▀▀▄▀█▄ ▀█▄▄▄▄█▀ █▀ ▀█▀██▄▀▄ ▄▄▀█▄████
█ ▄▄ ▄▄▄█▀ ▄ ▄██▄▀██▀█▄█▀▄█▀▄█ ▀██▀ █▄█
█ ▀█  ▄▄▄ ▀▄▄▀▀▀  ▄▀ ▀▄ ▀█▄▀▀▀ ▄▀▄  ▀██
██▄▄ ▄ ▄█▄ ▀▄▀▀█▄██▄▄█▄ ▄ ██ ▄▄█▄▀▀█▄▄█
█▀▄  ██▄ ▀█▄█▄█  █▄█ █▀▀ ▀▄█▀▄ ▄   ▄▄██
█▀ ▄▄▀▀▄ █▄▀ ▄█▀ ██▀▀ ▄ ▀█▄▀▄█ ▄ ▄ ▄ ▄█
█▀▄ █  ▄█  ██▀    ▀ ▀ █ █▄▀█▀▀█  ▄ ▀███
█ ██▄█▄▄▄██ █▀  ▄▄▄▀█▀▄▀▀█▄█ █▄▀ ▄  ▄ █
█▄███▄█▄█▀█▄█▄█▄  ▄█▀ ▀▀▄█▄▄█ ▄▄▄  ▄ ▄█
█ ▄▄▄▄▄ █ ▄▄▀▄▄ ▄▄▄▀▄ █▀▄ █▀  █▄█ ▀▄█ █
█ █   █ ██▀█▄▀▀█▀█ ▀▀██ ▀▀█▄    ▄▄▄ ▀▀█
█ █▄▄▄█ █▀ ▄█▀ ▄ ▀▄█▀▀▄ ▀█▄█ ▄████▀▄▄ █
█▄▄▄▄▄▄▄█▄▄▄▄▄▄██▄███▄████▄██▄▄▄█▄███▄█

ton://connect/GzhvQ-zwLhYNxzlEPEf-hP63kgR_0_O8vXsM8mSqQ-0?endpoint=connect.tonhubapi.com

Connected to wallet at address: EQC7zjln0_fghMQg0A-ZhYFar3DU1bDW9A4Vi5Go5uu-tAHe
```

After scanning the QR code and authorizing in your Sandbox - you will be requested to sign a transaction inside of the Sandbox application. When done - you will see following output in command line:

```
markokhman$ yarn blueprint run
yarn run v1.22.11
? Choose file to use (Use arrow keys)
? Choose file to use deploy
? Which network do you want to use? 
? Which network do you want to use? testnet
? Which wallet are you using? 
? Which wallet are you using? Tonhub wallet
Connected to wallet at address: EQDU69xgU6Mj-iNDHYsWWuNx7yRPQC_bNZNCpq5yVc7LiPVJ
Contract deployed at address EQCS7PUYXVFI-4uvP1_vZsMVqLDmzwuimhEPtsyQKIcdeNPu
You can view it at https://testnet.tonscan.org/address/EQCS7PUYXVFI-4uvP1_vZsMVqLDmzwuimhEPtsyQKIcdeNPu
✨  Done in 19.84s.
```

Congratulations! You've learned one more way to deploy your contracts :) And it's quite an effecient one.