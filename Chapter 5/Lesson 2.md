# Chapter 5. Lesson 2. Setting up TON Connect.

###### tags: `Chapter 5`

The TON Connect React library will provide us with some useful services like:
- showing the end-user a list of TON Connect 2 supported wallets
- querying the user's wallet for its public address 
- making a transaction request through to wallet

Install the library by running in terminal:
```
yarn add @tonconnect/ui-react
```

When our app connects to the user's wallet, we got to provide user with some ideantity information. "Who is asking for confirmation?". We will need to identify ourself using a manifest file. The wallet will ask for the user's permission to connect to the app and display the information from the manifest. 

Manifest file structure is looking like this:
```
{
  "url": "",
  "name": "Example",
  "iconUrl": ""
}
```

I will use a following contents for my `manifest.json` file:
```
{
  "url": "https://join.toncompany.org",
  "name": "TON&Co. Tutorial",
  "iconUrl": "https://raw.githubusercontent.com/markokhman/func-course-chapter-5-code/master/public/tonco.png"
}
```

Once I create a file **public/manifest.json**, put this contents there and push to the remote repository - my manifest will be availabel here:
```
https://raw.githubusercontent.com/markokhman/func-course-chapter-5-code/master/public/manifest.json
```

The manifest needs to be publicly available on the Internet, so you have to either follow my example and deploy your custom one, or for now - you can use an example one that I've deployed in advance during development. Later, when we deploy our website, you should replace the example manifest with your real one.

Modify the file `src/main.tsx` to use the TON Connect provider:

```
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';
import { TonConnectUIProvider } from '@tonconnect/ui-react';

// this manifest is used temporarily for development purposes
const manifestUrl = 'https://raw.githubusercontent.com/ton-community/tutorials/main/03-client/test/public/tonconnect-manifest.json';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <TonConnectUIProvider manifestUrl={manifestUrl}>
    <App />
  </TonConnectUIProvider>,
)
```

### Add a Connect button to the app

The first action we're going to offer the user is to Connect their wallet to the app. By connecting, the user agrees to share their public wallet address with the app. This isn't very dangerous since the entire transaction history of the wallet and its balance are publicly available on-chain anyways.

Edit the file `src/App.tsx` and replace its contents with the following:

```
import './App.css';
import { TonConnectButton } from '@tonconnect/ui-react';

function App() {
  return (
    <div>
      <TonConnectButton />
    </div>
  );
}

export default App
```

The only thing our new app UI will have is the Connect button. To run the app, run in command line:

```
npm run dev
```

Then refresh the web browser viewing the URL shown on-screen. I'm assuming you're using the web browser on your desktop (where you're developing) and your Tonkeeper wallet is installed on your mobile device. Click "Connect Wallet" on the desktop and choose "Tonkeeper" (or any other supporting wallet you're using).

TON Connect supports both mobile-mobile user flows and desktop-mobile user flows. Since development is a desktop-mobile flow, TON Connect will rely on scanning QR codes in order to communicate with the wallet running on your mobile device. Open the Tonkeeper mobile app, tap the QR button on the top right and scan the code from your desktop screen.

If everything is wired properly, you should see a confirmation dialong in the wallet mobile app. If you approve the connection, you will see your address in the web app UI!



**P.S. Credits for inspiration on lot's of code in this tutorial go to awesome [Shahar Yakir](https://ton-community.github.io/tutorials/03-client/) from [Orbs](https://www.orbs.com/) team.**
