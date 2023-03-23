# Chapter 5. Lesson 1. Dapp structure and basic project setup.

###### tags: `Chapter 5`

In previous chapter, we've created a smart contract with counter functinality as well as deposit/withdrawal features.  This smartcontract functions as the backend infrastructure for our application. In this tutorial, we will develop the frontend or user interface, enabling end-users to access it through a web browser.

It's important to remember that the application we're constructing is decentralized. Decentralized applications (DApps) follow specific rules. 
- One rule is that the backend must consist solely of smart contracts, without relying on any centralized servers. 
- Another rule is that the frontend must run exclusively on the client-side and be open-source. 

There are more, but as of now, those two are quite important to follow. By adhering to these rules, we ensure that we do not depend on a backend server to deliver our frontend. If such a server were utilized, it would inherently be centralized (since not all end-users would have equal access), and as a result, our entire application would become centralized too.

In this chapter we will work with another TON wallet called Tonkeeper. This will give you better understanding of how different wallets operate. The wallet will communicate with our DApp via a protocol called TON Connect 2. If you choose a different wallet than Tonkeeper, please verify it supports this version of TON Connect. Don't forget to make sure your wallet is connected to the correct network - mainnet or testnet.

> Tonkeeper works by default on TON mainnet. If you decided to work on testnet, you will need to switch the app manually to dev mode. Open the "Settings" tab and tap 5 times quickly on the Version number on the bottom. The "Dev Menu" should show up. Click on "Switch to Testnet" and make the switch. You can use this menu later to return to mainnet.

### TON Connect

Let's try to understand why do we even need TON Connect when we decide to create a user interface for our contract.

We can easily spot that there are 3 parties involved:

**TON blockchain** enables creation of trust-minimized applications and services at a massive scale.

**Apps** provide the UI to an infinite range of functionality in TON based on smart contracts, but do not have immediate access to users’ funds. 

**Wallets** provide the UI to approving transactions and hold users’ cryptographic keys securely on their personal devices.

This separation of concerns enables rapid innovation and high level of security for the users: wallets do not need to build walled-garden ecosystems themselves, while the apps do not need to take the risk holding end users’ accounts.

**TON Connect** is a bridge that crosses this conceptual gap.

To summarize, **TON Connect** is meant to streamline the way **Apps** interact with **TON Blockhain** via end-user's **Wallets**.

You will sometimes see **TON Connect 2**, this is just the most up to date version of protocol. Stick to wallets that support TON **Connect 2.**

### Setting up a project for our DApp

We will build our frontend with [React](https://reactjs.org/). To create our project we will rely on [Vite](https://vitejs.dev/) and its React template. Choose a name for your project, for example `counter-front-end`, then open terminal and run the following:

```
yarn create vite counter-front-end --template react-ts
cd counter-front-end && yarn
```

We will need to install a few more packages that will allow us to interact with TON Blockchain. We've seen these packages in action in the previous chapters while writing tests and compile/deploy scripts. Run following commands in your command line:

```
yarn add ton ton-core ton-crypto
yarn add @orbs-network/ton-access
```

Last but not least, we will need to overcome ton library's reliance on Nodejs Buffer that isn't available in the browser. We can do that by installing a polyfill. Run the following in terminal:
```
yarn add vite-plugin-node-polyfills
```

And to finish this Buffer fix, update your `vite.config.ts` file:
```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { nodePolyfills } from 'vite-plugin-node-polyfills';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), nodePolyfills()],
  base: '/',
});
```

To see your empty app template, run in command line:

```
yarn dev
```
Then open a web browser and direct it the URL shown in CLI results (something like http://localhost:5173/).


You will see a brand new page with contents of Vite's TypeScript template.

Voila! We're ready to start working with TON Connect to authenticate the end user for working with our DApp.


P.S. Credits for inspiration on parts of this tutorial go to awesome [Shahar Yakir](https://ton-community.github.io/tutorials/03-client/) from [Orbs](https://www.orbs.com/) team.
