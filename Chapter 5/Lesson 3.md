# Chapter 5. Lesson 3. Reading and presenting data from our contract

###### tags: `Chapter 5`

At this point we have implemented the functionality of end-user authorizing in order to interact with our contract.

Let's now actually connect our DApp to our actuall contract we deployed in Chapter 5.

There is a few requirements you need to take care of, in order to work properly with the steps we have to go through in this and upcoming lessons:

1. We are going to work with **tesnet**. Please make sure you've deployed your contract in Chapter 4 / Lesson 5 to testnet.
2. Make sure that in Chapter 4 / Lesson 5 you've deployed your contract with proper initial data, namely:
    * counter value
    * recent sender address
    * owner address - **this must be your active testnet wallet address**, because you are going to perform withdrawal operations and as you remember, the logic of our contract will deny any attempts to withdraw funds that are going to be made by non-owners.
3. Make sure you've got **testnet** coins. You can easily get them [here](https://t.me/testgiver_ton_bot).


At this point, I assume you have your contract from Chapter 4 / Lesson 5 deployed to testnet and you know it's address.

Here is the address of my contract:
```
EQCS7PUYXVFI-4uvP1_vZsMVqLDmzwuimhEPtsyQKIcdeNPu
```

As you remember, we've already done some ineractactions with our contract while writing tests. We had to create a wrapper file, that was helping us to send internal messages to our contract and also run getter methods. Pleasant surprise - we can use the same wrapper file :) 

Let's create a new folder `contracts` in our `src` folder and copy the file `wrappers/MainContract.ts` from our previous chapter. You can also find it's contents [here in my published repository](https://github.com/markokhman/func-course-chapter-4-code/blob/master/wrappers/MainContract.ts).

The contract wrapper file for our DApp is now located at `src/contracts/MainContract.ts`.

Now we have a proper wrapper for the contract, but how do we actually establish connection with an onchain contract from browser? In Chapter 3 we were detecting whether the contract is deployed or not and we used on instance of **TonClient** from **ton** library. It was using a **ton-access** RPC provider to retreive information from chain (thanks to [Orbs team](https://www.orbs.com/)) and send requests to our contract. We are going to use very same logic in this lesson.

Before we start, let's do "very React" thing, we will implement a general purpose React hook that will assist us in initializing async objects. We will use it to connect with **TonClient**.

Create the file `src/hooks/useAsyncInitialize.ts` with the following content:

```
import { useEffect, useState } from "react";

export function useAsyncInitialize<T>(
  func: () => Promise<T>,
  deps: any[] = []
) {
  const [state, setState] = useState<T | undefined>();
  useEffect(() => {
    (async () => {
      setState(await func());
    })();
  }, deps);

  return state;
}

```


Next, we're going to create another React hook that will rely on **useAsyncInitialize** and will initialize an RPC client in our app. As we've mentioned above, we are going to use [TON Access](https://github.com/orbs-network/ton-access) that will provide us with unthrottled API access for free. It's also decentralized, which is the preferred way to access the network.

Create the file `src/hooks/useTonClient.ts` with the following content:
```
import { getHttpEndpoint } from '@orbs-network/ton-access';
import { TonClient } from 'ton';
import { useAsyncInitialize } from './useAsyncInitialize';

export function useTonClient() {
  return useAsyncInitialize(
    async () =>
      new TonClient({
        endpoint: await getHttpEndpoint({ network: 'testnet' }),
      })
  );
}
```

Let's create one more hook called **useMainContract()** that is going to accept an on-chain address of a deployed contract and will run a getter method of our contract (with help of **.getData()** method of our wrapper).

This hook's code will look like this:

> Please be sure to replace the contract address with the one of your deployed contract

```
import { useEffect, useState } from "react";
import { MainContract } from "../contracts/MainContract";
import { useTonClient } from "./useTonClient";
import { useAsyncInitialize } from "./useAsyncInitialize";
import { Address, OpenedContract } from "ton-core";

export function useMainContract() {
  const client = useTonClient();
  const [contractData, setContractData] = useState<null | {
    counter_value: number;
    recent_sender: Address;
    owner_address: Address;
  }>();

  const mainContract = useAsyncInitialize(async () => {
    if (!client) return;
    const contract = new MainContract(
      Address.parse("EQCS7PUYXVFI-4uvP1_vZsMVqLDmzwuimhEPtsyQKIcdeNPu") // replace with your address from tutorial 2 step 8
    );
    return client.open(contract) as OpenedContract<MainContract>;
  }, [client]);

  useEffect(() => {
    async function getValue() {
      if (!mainContract) return;
      setContractData(null);
      const val = await mainContract.getData();
      setContractData({
        counter_value: val.number,
        recent_sender: val.recent_sender,
        owner_address: val.owner_address,
      });
    }
    getValue();
  }, [mainContract]);

  return {
    contract_address: mainContract?.address.toString(),
    ...contractData,
  };
}

```


At this point we are ready to present data we've read from chain onto our interface. Let's update our **src/App.tsx** with to have following code:

```
import "./App.css";
import { TonConnectButton } from "@tonconnect/ui-react";
import { useMainContract } from "./hooks/useMainContract";

function App() {
  const {
    contract_address,
    counter_value,
    recent_sender,
    owner_address,
    contract_balance,
  } = useMainContract();
  return (
    <div>
      <div>
        <TonConnectButton />
      </div>
      <div>
        <div className='Card'>
          <b>Our contract Address</b>
          <div className='Hint'>{contract_address?.slice(0, 30) + "..."}</div>
          <b>Our contract Balance</b>
          <div className='Hint'>{contract_balance}</div>
        </div>

        <div className='Card'>
          <b>Counter Value</b>
          <div>{counter_value ?? "Loading..."}</div>
        </div>
      </div>
    </div>
  );
}

export default App;

```

Great, at this point we are all set and our contract's data is easily read from the chain. Note that it can be read even if we didn't authorize with TON Connect, because this is read-only, we only need the authorization once we want to perform some "write" kind of actions.

In the next lesson we are going to perform those interactive actions to send increment, deposit and withdrawal transactions.


**P.S. Credits for inspiration on lot's of code in this tutorial go to awesome [Shahar Yakir](https://ton-community.github.io/tutorials/03-client/) from [Orbs](https://www.orbs.com/) team.**
