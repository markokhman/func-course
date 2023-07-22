## Chapter 4. Lesson 6. Verifying contract's source code
###### tags: `Chapter 4`

Now our contract deployed to the mainnet and we can verify the contract code.

From the explorer we see that our [code](link) is not verified. But here we can see an [example](link) of a verified one and anyone can see the logic of 
this contract in explorer. There is a new service in the TON ecosystem called [TON VERIFIER](link) and it will help us with it.
And this service is used to upload your source code and your source code is going to be compiled, code cell will be formatted, and the hash of the cell will be the proof that the code didn't change, basically. This code and this hash cell is stored in the decentralized way. Any explorer can actually retrieve the source code and be sure that the code didn't change before they actually retrieved it. There is no someone who is guaranteeing the code consistency because we don't need that. We actually have it in a decentralized way. 


### Using TON VERIFIER to verify contract code

Things we need to do:
1. Connect TonKeeper wallet by scanning QR-code.
2. Get the address of our [contract](link) deployed to mainnet. 
3. Provide TON VERIFIER with source FunC code of our contract (main.fc).
4. Also provide stdlib.fc and don't forget to specify the directory **imports** for it.
5. Click **Compile**.
6. Click **Publish**.
7. Wallet is asking you to confirm transaction and after confirming our verification is published.

We can check that now in explorer (tonscan) our contract is verified and after couple of minutes our code will be available for reading.
Also we can check our contract code in [TON API explorer](link).

### Deploying contract to testnet

First, deploy our contract to testnet with the command:
``` 
yarn blueprint run 
```

1. Choose testnet
2. Choose Tonkeeper
3. Authorize with the help of QR-code
4. Confirm transaction

After it we change **address** and **owner_address** in **deploy.ts** to our address in testnet.

```
    const myContract = MainContract.createFromConfig(
        {
            number: 0,
            address: address(""),
            owner_address: address(
                ""
            ),
        },
        codeCell
    );
```

Now deploy our contract to testnet the same way but with updated addresses.

Save this address for next lessons and we will build frontend for it.



