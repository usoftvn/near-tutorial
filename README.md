# HOW TO START WITH NEAR BLOCKCHAIN
If you don't know anything about NEAR, you should read this article. It helps you step by step approach to the NEAR blockchain and build a DApp on it.

# Appendix
1. Step 1: Learn the basics of NEAR Blockchain
2. Step 2: Prepare the development environment
3. Step 3: Know how to use some basic tools
4. Step 4: Create the first NEAR project
5. Step 5: Understanding how to write a smart contract using AssemblyScript
6. Step 6: Build, test and deploy a smart contract on NEAR Testnet blockchain
7. Step 7: Understanding how to write the frontend
8. Step 8: Deploy the frontend
9. Step 9: More resources

# Step 1: Learn the basics of NEAR Blockchain
First you need to understand some basics of NEAR blockchain:
- NEAR is a 3rd generation blockchain, born with the purpose of focusing on solving the problem of scalability using Sharding technology. Please see detail here: https://learnnear.club/what-is-near-protocol/
- NEAR uses a PoS consensus algorithm called Nightshade: https://near.org/papers/nightshade/
- Consensus protocols are used to reach agreement on a single value between multiple participants in a system. If all network participants collaborate in accordance with the consensus protocol, new values may be appended to the ledger and verified by nodes. However, in the case of disputes, the network may focus on either safety or liveness.

# Step 2: Prepare the development environment 
To be able to start developing on the NEAR blockhain, you need to prepare the environment for development:
- **NodeJs**: You need to install NodeJs version 14 or higher. Please download NodeJs from here: https://nodejs.org/en/
- **Yarn**: Install yarn using below command:
```
npm i -g yarn
```
- **near-sdk-as**: We will write the contract in AssemblyScript language, so we need to install near-sdk-as with the following command:
```
yarn add -D near-sdk-as
```
- **near-cli**: A tool that provides an interface for manipulating the NEAR blockchain. We use it to deploy Smart Contract and call Smart Contract functions
```
npm i -g near-cli
```
- **near-api-js**: 
If you need to program on NodeJs to connect to the NEAR Blockchain, the near-api-js library is essential. Before using on NodeJs you must install this library with command:
```
npm install near-api-js
```

If you know how to program at least 1 of NodeJs, Javascript, RUST and AsemblyScript languages, there will be more advantages.

# Step 3: Know how to use some basic tools
You should take a look at how to use the following tools:
- **NEAR Wallet**: Let's start by creating a first address on the NEAR Testnet https://wallet.testnet.near.org/. Next use the features on this website.
- **NEAR Explorer**: This is the place to help us look up transaction information to an account on the NEAR blockchain visually (https://explorer.testnet.near.org/)
- **near-sdk-core**: You will have to use it to look up functions that work with the blockchain. Please see detail here: https://near.github.io/near-sdk-as/globals.html
- **near-cli**: This is the tool you will have to work with regularly during the deployment and testing of smart contracts on NEAR. So you need to know some basic commands:
```
// Show help
near --help

// Login to your NEAR account. You can later use this account to deploy smart contracts.
// For example, I log in to my daothang.testnet account
near login

// Create a sub account, change daothang.testnet to your account
near create-account test01.daothang.testnet --masterAccount daothang.testnet --initialBalance 20

// Deploy smart contract to sub account
near deploy --accountId test01.daothang.testnet --wasmFile out/example.wasm
near deploy --accountId test01.daothang.testnet --wasmFile out/example.wasm --initFunction new --initArgs '{"owner_id": "example-contract.testnet", "total_supply": "10000000"}'

// Call a view function on smart contract (No fee)
near view test01.daothang.testnet getMessages '{}'

// Call a function on smart contract (Have fee)
near call test01.daothang.testnet addMessage '{"text": "Aloha"}' --account-id example-acct.testnet

// ...
```
Please see detail here: https://docs.near.org/docs/tools/near-cli

# Step 4: Create the first NEAR project
Now we will create the first project on NEAR with the following command:
```
// Create project hello-world
yarn create near-app hello-world

// Go to the project folder
cd hello-world
```
This command will create a project with a smart contract and frontend available. The image below is the guide information after the project is created:
<br />![Create NEAR Project using near-create-app](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-01.png)

Folder structure of the project:
- **contract**: Contain source code of the contract
- **src**: Contain source code of the frontend.
- **neardev**: Where to save account information used for development
- **package.json**: File save project information 
<br />![The project folder structure](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-05.png)

If you want to know more commands, please see at https://github.com/near/create-near-app

Now, we run it with the following command:
```
yarn dev
```
Build and run successfully you will see:
<br />![Build and run successfully](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-02.png)
And the browser automatically opens the link http://localhost:1234/ and you will see the following interface:
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-03.png)
You login using your wallet, you will see the main interface as follows:
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-04.png)
At this point, let's try to use the function of the application.


# Step 5: Understanding how to write a smart contract using AssemblyScript
Now let's focus on the **contract** folder, there are two important files:
- **contract/assembly/index.ts**: The file containing the source code of the contract.
- **contract/assembly/__tests__/main.spec.ts**: Unit test file for smart contract

Open the file source code **contract/assembly/index.ts**, you will see that our smart contract is quite simple, consisting of only 2 functions: 
- **setGreeting()** function: Save a message to the NEAR blockchain. Each account can only save 1 message. 
- **getGreeting()** function: Get message information for each account 
```
/*
 * This is an example of an AssemblyScript smart contract with two simple,
 * symmetric functions:
 *
 * 1. setGreeting: accepts a greeting, such as "howdy", and records it for the
 *    user (account_id) who sent the request
 * 2. getGreeting: accepts an account_id and returns the greeting saved for it,
 *    defaulting to "Hello"
 *
 * Learn more about writing NEAR smart contracts with AssemblyScript:
 * https://docs.near.org/docs/develop/contracts/as/intro
 *
 */

import { Context, logging, storage } from 'near-sdk-as'

const DEFAULT_MESSAGE = 'Hello'

// Exported functions will be part of the public interface for your smart contract.
// Feel free to extract behavior to non-exported functions!
export function getGreeting(accountId: string): string | null {
  // This uses raw `storage.get`, a low-level way to interact with on-chain
  // storage for simple contracts.
  // If you have something more complex, check out persistent collections:
  // https://docs.near.org/docs/concepts/data-storage#assemblyscript-collection-types
  return storage.get<string>(accountId, DEFAULT_MESSAGE)
}

export function setGreeting(message: string): void {
  const accountId = Context.sender
  // Use logging.log to record logs permanently to the blockchain!
  logging.log(`Saving greeting "${message}" for account "${accountId}"`)
  storage.set(accountId, message)
}

```

You should pay attention to a few points as follows:
- In the smart contract, there are 3 namespaces in the library of near-sdk-as. To manipulate blockchain using AsemblyScript, you are not required to use **near-sdk-as** library.
- Use **Context.sender** to know who called the function 
- Use **logging.log** to record logs permanently to the blockchain.
- Use **storage** to store data permanently to the blockchain. There are also many other data types to store data on the blockchain. See details at: https://docs.near.org/docs/concepts/data-storage#assemblyscript-collection-types

The file **contract/assembly/__tests__/main.spec.ts** contains source code to test smart contract:
```
import { setGreeting } from '..'
import { storage, Context } from 'near-sdk-as'

describe('Greeting ', () => {
  it('should be set and read', () => {
    setGreeting('hello world')
    storage.get<string>(Context.sender)
  })
})
```

Now try to modify the source code for adding your own functions and use more types: **PersistentVector**, **PersistentSet**, **PersistentMap**, **PersistentUnorderedMap**, **PersistentDeque**.


# Step 6: Build, test and deploy a smart contract on NEAR Testnet blockchain
We use the following command to test only the smart contract:
```
npm run build:contract:debug && cd contract && npm run test
```
You will the test run OK:
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-08.png)

We use the following command to build the smart contract:
```
yarn build:contract
```
The contract is built OK:
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-09.png)

Next we will create a sub account and deploy a smart contract to this sub account:
```
// Create sub account
near create-account helloworld.daothang.testnet --masterAccount daothang.testnet --initialBalance 10

// Deploy smart contract
near deploy --accountId helloworld.daothang.testnet --wasmFile out/main.wasm
```
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-10.png)

Let's try calling the functions on the smart contract:
```
near call helloworld.daothang.testnet setGreeting '{"message": "Good morning"}' --account-id daothang.testnet
near view helloworld.daothang.testnet getGreeting '{"accountId": "daothang.testnet"}'
```
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-11.png)

# Step 7: Understanding how to write the frontend
The entire frontend in the src directory, there are 4 files we need to care about:
- **src/config.js**: Contains network and contract configuration. In this file you change the contract to the new contract you just deployed.
<br />![](https://raw.githubusercontent.com/usoftvn/near-tutorial/main/images/near-tutorial-12.png)
- **src/utils.js**: Initialize and declare functions in contract.
- **src/index.js**: Contains the main javascript source code for business processing
- **src/index.html**: Main interface
- **src/main.test.js**: Used to test contract functions from users.

If you know javascript, it will be easy for you to read.

# Step 8: Deploy the frontend
To deploy the frontend, you run the following command:
```
yarn deploy
```
Then you can put all the files in the dist directory to some host to run.

# Step 9: More resources
In addition, you should read more in the following pages:
- **NEAR Account**: https://docs.near.org/docs/concepts/account
- **Near 101**: https://learn.figment.io/protocols/near
- **Near Example**: https://examples.near.org/
- **AssemplyScript**: https://www.assemblyscript.org/introduction.html
- **Hackathon Startup Guide**: https://docs.near.org/docs/develop/basics/hackathon-startup-guide
- **NEAR Tutorials**: https://docs.near.org/docs/tutorials/overview
