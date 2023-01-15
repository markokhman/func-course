# Chapter 3. Lesson 5.

We are on the finish line! Believe me, if you've made it this far - I'm more then sure you'll become a TON contracts developer. It's an honor for me to be the one who onboards you in this process!

So far, we have built our compilation and testing suits, but just like stallites are meant to be in Space - smartcontracts are meant to be on-chain. Let's go for it!

### Working with testnet

You've already heard of testnet, which is a separate network meant for testing all possible things for developers.

Testnet operates with test Ton coins. Those are not worth real value, so you don't need to buy them. You can easily get them from an official [testnet giver Telegram bot](https://t.me/testgiver_ton_bot). Initially, we are going to deploy or contract to testnet.

This bot will ask you for your address, so you need to have a wallet that would operate with test TON coins. You're probably familiary with wallets like Tonhub and Tonkeeper. Tonkeeper has the testnet functinality embed, while Tonhub's testnet functionality is separated into other mobile app called Sandbox. You can use both, Sandbox (by Tonwhales) or Tonkeeper, but for example purposes I will use the Sandbox wallet made by TonWhales team. It is awailable for both [iOS](https://apps.apple.com/tt/app/ton-development-wallet/id1607857373) and [Android](https://play.google.com/store/apps/details?id=com.tonhub.wallet.testnet&hl=en&gl=US). Follow one of the links and download it to your mobile.

Once you have installed Sandbox and created a wallet there - use it's address to receive testnet coins on [testnet giver Telegram bot](https://t.me/testgiver_ton_bot).

### Deploy script

Now let's create a new file in our **scripts** folder - **deploy.ts**. As you remeber, to deploy the contract we need to:

1. Calculate the future address of the contract
2. Compose a message that will have code and initial data in it
3. Send the composed message to the future addres of the contract on blockchain

Here is what our deploy script looks like before we actually send the deploy message:

```
import {hex} from "../build/main.compiled.json";
import {Cell, contractAddress, StateInit} from "ton";

async function deployScript() {

  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const initCell = new Cell();
  new StateInit({
    code: codeCell,
    data: dataCell,
  }).writeTo(initCell);

  const address = contractAddress({
    workchain: 0,
    initialCode: codeCell,
    initialData: dataCell,
  });

}

deployScript();
```

We're already familiar with all the types and entities except one - StateInit. StateInit is simply a helping wrapper for the Message type that we've used while writing our tests in the previous lesson. It accepts code and data, composes a message body and then we write the results into a cell that we are going to use while sending the deploy message.

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
import { Cell, contractAddress, StateInit, toNano } from "ton";
import qs from "qs";
import qrcode from "qrcode-terminal";

import dotenv from "dotenv";
dotenv.config();

async function deployScript() {
  const codeCell = Cell.fromBoc(Buffer.from(hex, "hex"))[0];
  const dataCell = new Cell();

  const initCell = new Cell();
  new StateInit({
    code: codeCell,
    data: dataCell,
  }).writeTo(initCell);

  const address = contractAddress({
    workchain: 0,
    initialCode: codeCell,
    initialData: dataCell,
  });

  let link =
    `https://test.tonhub.com/transfer/` +
    address.toFriendly({ testOnly: true }) +
    "?" +
    qs.stringify({
      text: "Deploy contract",
      amount: toNano(1).toString(10),
      init: initCell.toBoc({ idx: false }).toString("base64"),
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
import { Cell, contractAddress, StateInit, toNano } from "ton";
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

  const initCell = new Cell();
  new StateInit({
    code: codeCell,
    data: dataCell,
  }).writeTo(initCell);

  const address = contractAddress({
    workchain: 0,
    initialCode: codeCell,
    initialData: dataCell,
  });

  console.log(
    `The address of the contract is following: ${address.toFriendly()}`
  );
  console.log(`Please scan the QR code below to deploy the contract:`);

  let link =
    `https://test.tonhub.com/transfer/` +
    address.toFriendly({ testOnly: true }) +
    "?" +
    qs.stringify({
      text: "Deploy contract",
      amount: toNano(1).toString(10),
      init: initCell.toBoc({ idx: false }).toString("base64"),
    });

  qrcode.generate(link, { small: true }, (code) => {
    console.log(code);
  });
}

deployScript();
```

And again, we're immediately creating another script running shortcut in our **package.json** file:

```
{
  ... our previous package.json keys
  "scripts": {
    ... previous scripts keys
    "deploy": "ts-node ./scripts/deploy.ts"
  }
}
```

Let's run the deploy script (`yarn deploy`) and scan the QR code with the Sandbox mobile app. Make sure you have your tesnet TON coins on this wallet in Sanbox.

Once done, how do we check the contract is successfully deployed? Easily! You're probably already familiar with a concept of blockchain explorers. On of the go-to blockchain explorer in TON is [Tonscan](https://tonscan.org/). It also has a [testnet version](https://testnet.tonscan.org/).

Open the tesnet version in your browser and insert the address of the smartcontract into the search bar. This will open a page of your smartcontract. If you've done everything with me step by step - you will see a single transaction of 1 ton that has deployed our contract to the blockchain.

Congratulations! Your first TON smartcontract is deployed!

### Post-deploy onchain tests

### Deploying to mainnet
