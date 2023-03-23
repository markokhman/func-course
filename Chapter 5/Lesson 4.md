# Chapter 5. Lesson 4. Making increment, deposit and withdrawal transactions

###### tags: `Chapter 5`


The last interaction was read-only, let's interact with the contract by sending a transaction. 

The first feature we've developed on our contract is increment. Let's add a button to the main screen that sends this transaction. As you recall, sending a transaction on-chain costs gas, so we would expect the wallet to approve this action with the user and show how much TON coin will be spent.

Before starting, we're going a add another hook that will generate a sender object from the TON Connect interface. This sender represents the connected wallet and will allow us to send transactions on their behalf. 

While we're working on this transaction request, we'll also show in UI the wallet connection state so our UI would hide elements that require authorisation.

Just like we did in previous lessons, let's use React hook to keep some functionality that we are going to use in our code later on.

Let's create the file **src/hooks/useTonConnect.ts** with the following content:
```
import { useTonConnectUI } from '@tonconnect/ui-react';
import { Sender, SenderArguments } from 'ton-core';

export function useTonConnect(): { sender: Sender; connected: boolean } {
  const [tonConnectUI] = useTonConnectUI();

  return {
    sender: {
      send: async (args: SenderArguments) => {
        tonConnectUI.sendTransaction({
          messages: [
            {
              address: args.to.toString(),
              amount: args.value.toString(),
              payload: args.body?.toBoc().toString('base64'),
            },
          ],
          validUntil: Date.now() + 5 * 60 * 1000, // 5 minutes for user to approve
        });
      },
    },
    connected: tonConnectUI.connected,
  };
}
```

You can see that we're returing an object with a function **send** that will enable us to trigger a transaction request and parameter **connected** that stores a current state of the connection.

The next thing we're going to do is improving our existing **useCounterContract** hook and add two small features. 
1. Automatic polling of the counter value every 5 seconds. This will come in handy to show the user that the value indeed changed. 
2. Exposing the sendIncrement of our interface class and wiring it to the sender.


We need to update our code with a few things:

```
// ... previous imports
import { toNano } from "ton-core";
import { useTonConnect } from "./useTonConnect";


export function useCounterContract() {

    // ... variables definitions
    const { sender } = useTonConnect();
    
    const sleep = (time: number) => new Promise((resolve) => setTimeout(resolve, time));
    
    // ... mainContract definition
    
    useEffect(() => {
        async function getValue() {
    
          // ... previous getValue code

          await sleep(5000); // sleep 5 seconds and poll value again
          getValue();
        }
        getValue();
    }, [counterContract]);


    return {
        value: val,
        address: counterContract?.address.toString(),
        sendIncrement: () => {
          return mainContract?.sendIncrement(sender, toNano(0.05), 3);
        },
    };
}
```


At this point we only need to update our UI and it will be ready to trigger needed transacitons. We'll need to add two more things into it:

```
// ... previous imports
import { useTonConnect } from "./hooks/useTonConnect";


function App() {
    
    const { ..., sendIncrement } = useMainContract();
    const { connected } = useTonConnect()
    
    return (
        // ... other HTML code
            
            {connected && (
              <a
                onClick={() => {
                  sendIncrement();
                }}
              >
                Increment
              </a>
            )}
        
        // ... other HTML code
    )
}

```


We're all good, our functionality of sending Increment is working just fine :)


