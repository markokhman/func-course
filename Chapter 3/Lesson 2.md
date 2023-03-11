# Chapter 3. Lesson 2.

###### tags: `Chapter 3`

The goal of this lesson is to setup a local project that is able to compile a sample smartcontract. We are not going to write any FunC just yet. We are just preparing our laboratory. This part can be reused across all your projects later on.

First of all, please make sure that you have three things:

- **A modern version of Node.js** (version 16.15.0 or later)
  Installation instructions can be found [here](https://nodejs.org/)
  Run command `node -v` in terminal to verify your installation
- **A package manager** - It's probably is already installed together with Node.js, but please make sure that you have one. In this lesson we are going to use **Yarn**, but you can use one of your choice (ex. **npm**)
- **A decent IDE with FunC and TypeScript** support
  We recommend using [Visual Studio Code](https://code.visualstudio.com/) with the [FunC plugin](https://marketplace.visualstudio.com/items?itemName=tonwhales.func-vscode) installed. In case you are using IntelliJ, here is the link to a corresponding [FunC plugin](https://plugins.jetbrains.com/plugin/18541-ton-development)

Once the above mentioned dependencies are met - we're ready to start!

### Setting up a project.

Let's first create a folder for our project

```
mkdir my_first_contract && cd my_first_contract
```

Let's initiate a **package.json** file with a package manager. I'm going to use **yarn**, but if you are feeling better to go with npm - you're free to choose.

```
yarn init
```

You will be prompted to enter a number of parameters, but feel free to simply click Enter button on every prompt. Once done, out project's directory is supposed to have package.json file with following default contents:

```
{
  "name": "my_first_contract",
  "version": "1.0.0",
  "main": "index.js", //we don't really need this line, feel free to remove it
  "license": "MIT"
}
```

Now, let's install following 4 libraries:

> Those are TypeScript related libraries. In this course we are not going to dig into details of how TypeScript works itself.

```
yarn add typescript ts-node @types/node @swc/core --dev
```

Let's create a **tsconfig.json** file in a root of our project and put following config there:

```
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  },
  "ts-node": {
    "transpileOnly": true,
    "transpiler": "ts-node/transpilers/swc"
  }
}

```

Three more libs that we will need are actually TON related, those are:

- **ton-core** - Core library that implements low level primitives of TON blockchain.
- **ton-crypto** - Crypto primitives for building apps for TON blockchain.
- **@ton-community/func-js** - TON FunC compiler.

```
yarn add ton-core ton-crypto @ton-community/func-js --dev
```

### Sample FunC code

Now we are going to create a file with sample minimal FunC code and then write a script that would compile it.

Let's create a **contracts** folder and one FunC file (main.fc) in it:

```
mkdir contracts && cd contracts && touch main.fc
```

Open the **main.fc** file for editing and insert following sample FunC code:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {

}
```

> You're already aware that TON Smartcontract is able to recieve two kinds of messages, please refer to the Chapter 1, in case you're not sure what are those messages.

This simple code will be enough for us to write a compiler script. Let's go for it!

### Writing our compile script

Create a folder **scripts** in the root of our project and new file **compile.ts** in **scripts** folder:

```
mkdir scripts && cd scripts && touch compile.ts
```

We have a **compile.ts** file, but let's make it comfy to run once we finish it's code. Let's create a script shortcut in the **package.json** file. Just add following **scripts** key:

```
{
   //...your previous package.json contents
   "scripts": {
	"compile": "ts-node ./scripts/compile.ts"
   }
}
```

Now open the **scripts/compile.ts** file in your editor and let's start writing our compile script. We are going to add code step by step along with explanations, so by the end of this section you will have a complete code for the compile script.

First of all we will import:

- **fs** for working with files
- **process** to controll the script execution process
- **Cell** constructor (the bytecode of our contract is going to be stored as Cell)
- **compileFunc** - actual compilation function

```
import * as fs from "fs";
import process from "process";
import { Cell } from "ton-core";
import { compileFunc } from "@ton-community/func-js";
```

We are going to have some asynchronous code, so let's create an async function that will run it inside:

```
import * as fs from "fs";
import process from "process";
import { Cell } from "ton-core";
import { compileFunc } from "@ton-community/func-js";

async function compileScript() {

}

compileScript();

```

Now let's pass our sample contract's filename into the the **compileFunc** function and exit script if the result status is error:

```
import * as fs from "fs";
import process from "process";
import { Cell } from "ton-core";
import { compileFunc } from "@ton-community/func-js";

async function compileScript() {

  const compileResult = await compileFunc({
    targets: ["./contracts/main.fc"],
    sources: (x) => fs.readFileSync(x).toString("utf8"),
  });

  if (compileResult.status === "error") {
    process.exit(1);
  }

}
compileScript();

```

Awesome! Technically, you could already go to the root of our project and run `yarn compile` command. But if you do so - you will not be able to see the results of the compilation. Let's save the results of our compilation into folder **build** in the root of our project (make sure you create it first).

The compilation result is going to be a BOC (Body of Cell) base64 string, but if we want to further work with this compiled contract locally (while writing tests), we need to construct a Cell that will contain this BOC and store it for later usage.

Cell constructor has a method **.fromBoc** that expects to receive a Buffer, so we will provide it a Buffer of the BOC base64 string. Once we constuct a Cell from BOC - we want to create a hexadecimal representation of this Cell and store it in a JSON file. This is how are Cell constructing and transforming command looks like:

`Cell.fromBoc(Buffer.from(compileResult.codeBoc, "base64"))[0].toBoc()
        .toString("hex")`

And now let's put it into our script, along with using **fs.writeFileSync** to save the results into our JSON file:

```
import * as fs from "fs";
import process from "process";
import { Cell } from "ton-core";
import { compileFunc } from "@ton-community/func-js";

async function compileScript() {

  const compileResult = await compileFunc({
    targets: ["./contracts/main.fc"],
    sources: (x) => fs.readFileSync(x).toString("utf8"),
  });

  if (compileResult.status === "error") {
    process.exit(1);
  }

  const hexArtifact = `build/main.compiled.json`;

  fs.writeFileSync(
    hexArtifact,
    JSON.stringify({
      hex: Cell.fromBoc(Buffer.from(compileResult.codeBoc, "base64"))[0]
        .toBoc()
        .toString("hex"),
    })
  );
}
compileScript();
```

This code is ready to be run, but let's do some final edits to make our script informative while it is running. We are going to add nice console logs on each step.

Here is the final code along with console logs:

```
import * as fs from "fs";
import process from "process";
import { Cell } from "ton-core";
import { compileFunc } from "@ton-community/func-js";

async function compileScript() {
  console.log(
    "================================================================="
  );
  console.log(
    "Compile script is running, let's find some FunC code to compile..."
  );

  const compileResult = await compileFunc({
    targets: ["./contracts/main.fc"],
    sources: (x) => fs.readFileSync(x).toString("utf8"),
  });

  if (compileResult.status === "error") {
    console.log(" - OH NO! Compilation Errors! The compiler output was:");
    console.log(`\n${compileResult.message}`);
    process.exit(1);
  }

  console.log(" - Compilation successful!");

  const hexArtifact = `build/main.compiled.json`;

  fs.writeFileSync(
    hexArtifact,
    JSON.stringify({
      hex: Cell.fromBoc(Buffer.from(compileResult.codeBoc, "base64"))[0]
        .toBoc()
        .toString("hex"),
    })
  );

  console.log(" - Compiled code saved to " + hexArtifact);
}
compileScript();
```

This is our compile script, that we will run every time we do some changes to our code. Later on, once we will have some testable FunC logic and deploy scripts - we can automate running of this script before any tests or deploy.

Our "satellite assembly" laboratory is getting more and more equiped!

But let's run it one more time `yarn compile` and see what are the results of our compilation. For this, let's open our **main.compiled.fc** file.

What we see there:

```
{"hex":"b5ee9c72410102010012000114ff00f4a413f4bcf2c80b010006d35f03fbffbf07"}
```

Sounds crazy, but this is our contract. In the next lesson we are going to write our first FunC logic and compile it with the script we just wrote.

See you there!
