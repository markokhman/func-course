# Chapter 3. Lesson 5.

###### tags: `Chapter 3`

> As we've mentioned in our 1 Lesson of this chapter, TON has a rapidly growing set of programming tools, and sometimes it doesn't make sense to create your own custom local setup. There is a great one worth mentioning that is maintained by the TonTech team, officially supported by TON Foundation - [Blueprint](https://github.com/ton-community/blueprint).
Deploy process is also simplified very much in Blueprint. However we are going to create our custom deploy script to better understand the logic behind it.


We are on the finish line! Believe me, if you've made it this far - I'm more then sure you'll become a TON contracts developer. It's an honor for me to be the one who onboards you in this process!

So far, we have built our compilation and testing suits, but just like stallites are meant to be in Space - smartcontracts are meant to be on-chain. Let's go for it!

### Working with testnet

You've already heard of testnet, which is a separate network meant for testing all possible things for developers. 

Testnet operates with test Ton coins. Those are not worth real value, so you don't need to buy them. You can easily get them from an official [testnet giver Telegram bot](https://t.me/testgiver_ton_bot). Initially, we are going to deploy or contract to testnet.

This bot will ask you for your address, so you need to have a wallet that would operate with test TON coins. You're probably familiary with wallets like Tonhub and Tonkeeper. Tonkeeper has the testnet functinality embed, while Tonhub's testnet functionality is separated into other mobile app called Sandbox. You can use both, Sandbox (by Tonwhales) or Tonkeeper, but for example purposes I will use the Sandbox wallet made by TonWhales team. It is available for both [iOS](https://apps.apple.com/tt/app/ton-development-wallet/id1607857373) and [Android](https://play.google.com/store/apps/details?id=com.tonhub.wallet.testnet&hl=en&gl=US). Follow one of the links and download it to your mobile.

Once you have installed Sandbox and created a wallet there - use it's address to receive testnet coins on [testnet giver Telegram bot](https://t.me/testgiver_ton_bot).

### Deploy script

Now let's create a new file in our **scripts** folder - **deploy.ts**. As you remeber, to deploy the contract we need to:
1. Calculate the future address of the contract
2. Compose an initial state of the contract (code and initial data)
3. Compose a message that will carry the initial state 
4. Send the composed message to the future addres of the contract on blockchain

Here is what our deploy script looks like before we actually send the deploy message:
```
import {hex} from "../build/main.compiled.json";
import {Cell, contractAddress, StateInit} from "ton";

async function deployScript() {

  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const stateInit: StateInit = {
    code: codeCell,
    data: dataCell,
  };

  const stateInitBuilder = beginCell();
  storeStateInit(stateInit)(stateInitBuilder);
  const stateInitCell = stateInitBuilder.endCell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });
  
}

deployScript();
```

We're already familiar with all the types and entities except one - **StateInit** type and **storeStateInit** function. **storeStateInit** is provided to us by `ton-core` library and it helps us to create a cell that would have StateInit inside of it.

Let me elaborate more on what happens on these lines:
```
const stateInit: StateInit = {
    code: codeCell,
    data: dataCell,
};

const stateInitBuilder = beginCell();
storeStateInit(stateInit)(stateInitBuilder);
const stateInitCell = stateInitBuilder.endCell();
```

An important concept that is used here is Builder. You will need to understand that to succesfully create FunC contracts and operate them with TypeScript. As you can see in the code, we are using **beginCell** function that returns us a Builder (you can see this if hover on the **stateInitBuilder** variable and check it's type).

We pass the **stateInit** into the **storeStateInit** - this gives us a function that is ready to write provided StateInit into a builder. We immediately execute this function and pass the builder that we created on previous step. 

At this moment we already have a builder that already has the desired InitState data written into it. All we have to do is to run it's **.endCell()** function and it will finalise the builder into a ready cell with StateInit in it.

We could achieve the same result by manually constructing such a cell. Here is the code:

```
  const stateInitCell = beginCell()
    .storeBit(false) // split_depth - Parameter for the highload contracts, defines behaviour of splitting into multiple instances in different shards. Currently StateInit used without it.
    .storeBit(false) // special - Used for invoking smart contracts in every new block of the blockchain. Available only in the masterchain. Regular user's contracts used without it.
    .storeMaybeRef(codeCell) // code - Contract's serialized code.
    .storeMaybeRef(dataCell) // data - Contract initial data.
    .storeUint(0, 1) // library - Currently used StateInit without libs
    .endCell();
```

However, this would require us to deeper understand the body of InitState. You can read more about it [here in documentation](https://ton.org/docs/develop/data-formats/msg-tlb#stateinit-tl-b).

### Composing a message with StateInit

In order to send a message, we are going to need the Sandbox wallet that you've installed in the begining of this lesson. Sandbox has cool API that enables deep linking into the wallet, meaning that we could compose a link, that would have all the parameters needed to send a message, then open this link in Sandbox and sign the actual transaction of sending a message. This will require an install of a **qs** library that will help us to compose url with sophisticated parameters.

Install it by running:
```
yarn add qs @types/qs --dev
```

Because our Sandbox wallet is on mobile and the link is on our desktop - let's just generate a qr code of this link and show it in terminal to scan with Sanbox and send the deploy message. We are going to use library **qrcode-terminal** for this purposes.

Install it by running:
```
yarn add qrcode-terminal @types/qrcode-terminal --dev
```

With help of those two libraries, our deploy script is now looking like this:

```
import { hex } from "../build/main.compiled.json";
import { beginCell, Cell, contractAddress, StateInit, storeStateInit, toNano } from "ton";
import qs from "qs";
import qrcode from "qrcode-terminal";

import dotenv from "dotenv";
dotenv.config();

async function deployScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const stateInit: StateInit = {
    code: codeCell,
    data: dataCell,
  };

  const stateInitBuilder = beginCell();
  storeStateInit(stateInit)(stateInitBuilder);
  const stateInitCell = stateInitBuilder.endCell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });

  let link =
    `https://test.tonhub.com/transfer/` +
    address.toString({
      testOnly: true,
    }) + "?" +
    qs.stringify({
      text: "Deploy contract",
      amount: toNano(1).toString(10),
      init: stateInitCell.toBoc({ idx: false }).toString("base64"),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });
}

deployScript();

```

As we've done in previous lessons, let's add informative console logs, because you might want to reuse those scripts across all you projects.

Now our code will look like this:

```
import { hex } from "../build/main.compiled.json";
import {
  beginCell,
  Cell,
  contractAddress,
  StateInit,
  storeStateInit,
  toNano,
} from "ton";
import qs from "qs";
import qrcode from "qrcode-terminal";

import dotenv from "dotenv";
dotenv.config();

async function deployScript() {
  console.log(
    "================================================================="
  );
  console.log("Deploy script is running, let's deploy our main.fc contract...");

  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const stateInit: StateInit = {
    code: codeCell,
    data: dataCell,
  };

  const stateInitBuilder = beginCell();
  storeStateInit(stateInit)(stateInitBuilder);
  const stateInitCell = stateInitBuilder.endCell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });

  console.log(
    `The address of the contract is following: ${address.toString()}`
  );
  console.log(`Please scan the QR code below to deploy the contract:`);

  let link =
    `https://test.tonhub.com/transfer/` +
    address.toString({
      testOnly: true,
    }) +
    "?" +
    qs.stringify({
      text: "Deploy contract",
      amount: toNano(1).toString(10),
      init: stateInitCell.toBoc({ idx: false }).toString("base64"),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });
}

deployScript();

```


And again, we're immediately creating another script running shortcut in our **package.json** file. Let's call this script **deploy:testnet**, so we can later use the name **deploy** for mainnet deployments:
```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "deploy:testnet": "yarn compile && ts-node ./scripts/deploy.ts"
  }
}
```

Let's run the deploy script (`yarn deploy:testnet`) and scan the QR code with the Sandbox mobile app. Make sure you have your tesnet TON coins on this wallet in Sanbox.

Once done, how do we check the contract is successfully deployed? Easily! You're probably already familiar with a concept of blockchain explorers. On of the go-to blockchain explorer in TON is [Tonapi](https://tonapi.io/). It also has a [testnet version](https://testnet.tonapi.io/). 

Open the tesnet version in your browser and insert the address of the smartcontract into the search bar. This will open a page of your smartcontract. If you've done everything with me step by step - you will see a single transaction of 1 ton that has deployed our contract to the blockchain.

Congratulations! Your first TON smartcontract is deployed!

### Post-deploy onchain test

Yes, you've made it, but there is an important thing that you really want to take care of - post-deploy on-chain test. Once our contract is deployed, we want to ensure the functinality behaves the way we expect it to. For this we will write a script that will actually test our functinality.

However, before running this test we need to make sure that our contract is deployed. To do so, we need to acces the current state of TON blockchain. There is a great tool that allows us to connect to almost every possible type of TON blockchain APIs - [TON Access](https://github.com/orbs-network/ton-access) assembled by [Orbs team](https://www.orbs.com/).

First things first - create a file **onchaintest.ts** in our **scripts** folder.

Add script running shortcut into the **package.json** file:
```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "onchaintest": "ts-node ./scripts/onchaintest.ts"
  }
}
```

To check if the contract is deployed we need it's address. We want our setup to be comfy to use and require minimal manual work, so we will not hardcode the address we've seen prior to deploy. We are going to calculate it independently, because we remember, that the address might change based on the contract code and it's initial state data.

This is how our initial code looks when we want to get an address of our contract:
```
import { Cell, contractAddress } from "ton";
import { hex } from "../build/main.compiled.json";

async function onchainTestScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });
}

onchainTestScript();

```

Let's first install the **[@orbs-network/ton-access](https://github.com/orbs-network/ton-access)** library:
```
yarn add @orbs-network/ton-access --save
```

Now let's update our code with a logic of checking our contract's state on the network by it's address:
```
import { Cell, contractAddress } from "ton";
import { hex } from "../build/main.compiled.json";
import { getHttpV4Endpoint } from "@orbs-network/ton-access";

async function onchainTestScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const address = contractAddress({
    workchain: 0,
    initialCode: codeCell,
    initialData: dataCell,
  });
  
  const endpoint = await getHttpV4Endpoint({
    network: "testnet",
  });
  const client4 = new TonClient4({ endpoint });

  const latestBlock = await client4.getLastBlock();
  let status = await client4.getAccount(latestBlock.last.seqno, address);

  if (status.account.state.type !== "active") {
    console.log("Contract is not active");
    return;
  }

}

onchainTestScript()
```

Let's dive a little bit more into this code. What have we got here? 
- we use **getHttpV4Endpoint** function to connect to an API endpoint hosted by Orbs team and get a client object of v4 protocol. You can learn more about v4 protocol [here](https://github.com/ton-community/ton-api-v4)
> So far we have to set the config parameter of network to **testnet**.
- we get information about last block
- we use v4 client's **getAccount** method, providing it the sequence number and our contract's **address**

As a result, we can stop our script in case the contract was not yet deployed.

Now let's get to the actual test. We are going to create a link for a simple transaction that will send 1 TON to our contract's address. Same as we did during the deploy process, we will show QR code of this link, so you could scan it with your Sandbox wallet mobile app.

```
let link =
    `https://test.tonhub.com/transfer/` +
    address.toString({
      testOnly: true,
    }) +
    "?" +
    qs.stringify({
      text: "Simple test transaction",
      amount: toNano(1).toString(10),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });
```

We have to also import **toNano** function from **ton-core** library. It converts amount of TON coins into nanos. One TON has 1 000 000 000 nanos in it. In TON, an amount of TON coins is always represented in nanos. For usability, most of the times we will operate with nanos as a string.

Also, note, that we are using the **toString** method of the adress object, providing it config parameter that specifies the address is from testnet.

Immediately after showing the QR code, our script will start invoking a getter method on our contract, to receive the latest sender's address. We are going to only console log if the the latest sender's address changes, to avoid consol logging same results.

Let's first see the code for this logic of getting the latest sender's, we will breakdown what happens there and then integrate it to our script:
```
let recent_sender_archive: Address;

  setInterval(async () => {
    const latestBlock = await client4.getLastBlock();
    const { exitCode, result } = await client4.runMethod(
      latestBlock.last.seqno,
      address,
      "get_the_latest_sender"
    );

    if (exitCode !== 0) {
      console.log("Running getter method failed");
      return;
    }
    if (result[0].type !== "slice") {
      console.log("Unknown result type");
      return;
    }

    let most_recent_sender = result[0].cell.beginParse().loadAddress();

    if (
      most_recent_sender &&
      most_recent_sender.toString() !== recent_sender_archive?.toString()
    ) {
      console.log(
        "New recent sender found: " +
          most_recent_sender.toString({ testOnly: true })
      );
      recent_sender_archive = most_recent_sender;
    }
  }, 2000);
```
What exactly do we do in this code:
- we are setting up this function to repeat every 2 seconds
- every time we perform a check, we need to get the latest block's sequence number with **getLastBlock**
- we're using **runMethod** to call our getter method and parameters we're interested in are **exitCode** and **result**.
- we make sure the script stops if the call was not successfull or the data received is not of a slice type (as you remember, an address in TON is always a slice)
- we take the result of get method, namely it's first item (the **result** is an array, as in other cases there could be more then one result returned by getter method)
- we take a **cell** parameter of the **result**, open it for parse and then read an address from it
- we compare the string version of the address to the string version of the possible previous sender we've received. If they are not equal - we console log the new **most recent sender's address**.

Let's see, how our final code looks:
```
import { Address, Cell, contractAddress, toNano } from "ton-core";
import { hex } from "../build/main.compiled.json";
import { getHttpV4Endpoint } from "@orbs-network/ton-access";
import { TonClient4 } from "ton";
import qs from "qs";
import qrcode from "qrcode-terminal";

async function onchainTestScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });

  const endpoint = await getHttpV4Endpoint({
    network: "testnet",
  });
  const client4 = new TonClient4({ endpoint });

  const latestBlock = await client4.getLastBlock();
  let status = await client4.getAccount(latestBlock.last.seqno, address);

  if (status.account.state.type !== "active") {
    console.log("Contract is not active");
    return;
  }

  let link =
    `https://test.tonhub.com/transfer/` +
    address.toString({
      testOnly: true,
    }) +
    "?" +
    qs.stringify({
      text: "Simple test transaction",
      amount: toNano(1).toString(10),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });

  let recent_sender_archive: Address;

  setInterval(async () => {
    const latestBlock = await client4.getLastBlock();
    const { exitCode, result } = await client4.runMethod(
      latestBlock.last.seqno,
      address,
      "get_the_latest_sender"
    );

    if (exitCode !== 0) {
      console.log("Running getter method failed");
      return;
    }
    if (result[0].type !== "slice") {
      console.log("Unknown result type");
      return;
    }

    let most_recent_sender = result[0].cell.beginParse().loadAddress();

    if (
      most_recent_sender &&
      most_recent_sender.toString() !== recent_sender_archive?.toString()
    ) {
      console.log(
        "New recent sender found: " +
          most_recent_sender.toString({ testOnly: true })
      );
      recent_sender_archive = most_recent_sender;
    }
  }, 2000);
}

onchainTestScript();
```

Let's go ahead and run it!

```
yarn onchaintest
```

If you've done everything right, you will see a QR code in your terminal and the most recent sender's address. It has to equal to the address of your Sandbox wallet address, the one you've used for deploying the contract. 

> When we were deploying the contract, we've sent the code and initial state along with 1 TON, so immediately after the contract was deployed, it's code got executed and saved the sender into c4 storage.

To make sure our script works as expected, you could create another wallet on Sanbox, fill it with tesnet TON coins and again send it to our contract. You would see an update in the console after a few seconds, because the most recent sender's address changed.

### Deploying to mainnet

Let's adjust our script, so we could specify whether we want to deploy the contract to testnet or the mainnet. This will require us two things:
1. We will introduce an environment variable **TESTNET**, that will be set depending on the script we are running (deploy or deploy_mainnet)
2. We have to update our code in all the places that include testnet/mainnet differentiation.

Let's first update our **package.json** file:
```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "deploy": "TESTNET=true ts-node ./scripts/deploy.ts",
    "deploy:mainnet": "ts-node ./scripts/deploy.ts",
    "onchaintest": "TESTNET=true ts-node ./scripts/onchaintest.ts",
    "onchaintest:mainnet": "ts-node ./scripts/onchaintest.ts"
  }
}
```
As you can see, we are now passing a **TESTNET** variable for testnet deploy, let's now update our code of **deploy.ts** file:

```
import { hex } from "../build/main.compiled.json";
import {
  beginCell,
  Cell,
  contractAddress,
  StateInit,
  storeStateInit,
  toNano,
} from "ton";
import qs from "qs";
import qrcode from "qrcode-terminal";

import dotenv from "dotenv";
dotenv.config();

async function deployScript() {
  console.log(
    "================================================================="
  );
  console.log("Deploy script is running, let's deploy our main.fc contract...");

  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const stateInit: StateInit = {
    code: codeCell,
    data: dataCell,
  };

  const stateInitBuilder = beginCell();
  storeStateInit(stateInit)(stateInitBuilder);
  const stateInitCell = stateInitBuilder.endCell();

  const address = contractAddress(0, {
    code: codeCell,
    data: dataCell,
  });

  console.log(
    `The address of the contract is following: ${address.toString()}`
  );
  console.log(`Please scan the QR code below to deploy the contract:`);

  let link =
    `https://${process.env.TESTNET ? "test." : ""}.tonhub.com/transfer/` +
    address.toString({
      testOnly: process.env.TESTNET ? true : false,
    }) +
    "?" +
    qs.stringify({
      text: "Deploy contract",
      amount: toNano(1).toString(10),
      init: stateInitCell.toBoc({ idx: false }).toString("base64"),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });
}

deployScript();
```

We only had to change our code in two places:
- when logging the contract address, we set **testOnly** flag only we're on testnet
- when composing a link for a deploy transaction


Now, let's update our **onchaintest.ts** script with the same:

```
import { Cell, contractAddress, Address, TonClient4, toNano } from "ton";
import { hex } from "../build/main.compiled.json";
import { getHttpV4Endpoint } from "@orbs-network/ton-access";
import qs from "qs";
import qrcode from "qrcode-terminal";

async function onchainTestScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const address = contractAddress({
    workchain: 0,
    initialCode: codeCell,
    initialData: dataCell,
  });

  const endpoint = await getHttpV4Endpoint({
    network: process.env.TESTNET ? "testnet" : "mainnet",
  });
  const client4 = new TonClient4({ endpoint });

  const latestBlock = await client4.getLastBlock();
  let status = await client4.getAccount(latestBlock.last.seqno, address);

  if (status.account.state.type !== "active") {
    console.log("Contract is not active");
    return;
  }

  let link =
    `https://${process.env.TESTNET ? "test." : ""}tonhub.com/transfer/` +
    address.toFriendly({ testOnly: true }) +
    "?" +
    qs.stringify({
      text: "Simple test transaction",
      amount: toNano(1).toString(10),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });

  let recent_sender_archive: Address;

  setInterval(async () => {
    const latestBlock = await client4.getLastBlock();
    const { exitCode, result } = await client4.runMethod(
      latestBlock.last.seqno,
      address,
      "get_the_latest_sender"
    );

    if (exitCode !== 0) {
      console.log("Running getter method failed");
      return;
    }
    if (result[0].type !== "slice") {
      console.log("Unknown result type");
      return;
    }

    let most_recent_sender = result[0].cell.beginParse().readAddress();

    if (
      most_recent_sender &&
      most_recent_sender.toString() !== recent_sender_archive?.toString()
    ) {
      console.log(
        "New recent sender found: " +
          most_recent_sender.toFriendly({
            testOnly: process.env.TESTNET ? true : false,
          })
      );
      recent_sender_archive = most_recent_sender;
    }
  }, 2000);
}

onchainTestScript();
```

We had to similarly change three things in our **onchaintest.ts** script:
- when we connect to the client in order to read blockchain
- when logging the contract address, we set **testOnly** flag only we're on testnet
- when composing a link for a deploy transaction

Now we can deploy and run onchain test on both - testnet and mainnet!

> **Keep in mind that for mainnet you need to use TonHub wallet (mainnet version of Sandbox). It is also available for [iOS](https://apps.apple.com/tt/app/tonhub-ton-wallet/id1607656232) and [Android](https://play.google.com/store/apps/details?id=com.tonhub.wallet&hl=en&gl=US).**

You've made it! Seriously, I'm proud of you! We've done a solid amount of work, but the good thing is that now you understand the whole process of how contracts get developed and deployed.

In our next chapter we are going to dig into more complex FunC logic and writing local tests for it.
