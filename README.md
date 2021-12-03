# HOW TO START WITH NEAR BLOCKCHAIN
If you don't know anything about NEAR, you should read this article. It helps you step by step approach to the NEAR blockchain and build a DApp on it.

# Appendix
1. Step 1: Learn the basics of NEAR Blockchain
2. Step 2: Prepare the development environment
3. Step 3: Know how to use some basic tools
4. Step 4: Create the first NEAR project
5. Step 4: Writing a smart contract using AssemblyScript
6. Step 5: Deploy a smart contract on NEAR Testnet blockchain
7. Step 6: Build the frontend

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

In addition, you should take a look in the following pages:
- **NEAR Account**: https://docs.near.org/docs/concepts/account
- **Near 101**: https://learn.figment.io/protocols/near
- **Near Example**: https://examples.near.org/
- **AssemplyScript**: https://www.assemblyscript.org/introduction.html
- **Hackathon Startup Guide**: https://docs.near.org/docs/develop/basics/hackathon-startup-guide

# Step 4: Create the first NEAR project
Now we will create the first project on NEAR with the following command:
```
yarn create near-app hello-world
cd hello-world
```
If you want to know more commands, please see at https://github.com/near/create-near-app
This command will create a project with a smart contract and frontend available.




, and it does many other things. You can see more details at NEAR-CLI. You use the following command to install:
=================================
1. Follow NEAR Hackaton guide to setup environment - https://docs.near.org/docs/develop/basics/hackathon-startup-guide
2. Clone or download this repository
3. Update `CONTRACT_NAME` in `src\config.js` with your NEAR account
4. Install dependencies with the command below

```
yarn install
```

# Smart Contract

Essentially, this is a program that runs on blockchain. We will go through my contract step by step after which you should be able to make your own modifications and deploy your version of Voting system.

# Lets Dig In!

Open up `assembly\main.ts` file right at the beginning I setup few data structures required by our Voting System.

## Candidate class

```
@nearBindgen
class Candidate {
	avatar: u64;
	voteCount: number;
	alive: boolean;

	constructor(public name: string) {
		this.avatar = context.blockIndex;
		this.voteCount = 0;
		this.alive = true;
	}
}
```

First line `@nearBindgen` is a special as it allows NEAR serialize our Candidate objects to be serialized on blockchain.

After that we define few properties (`avatar`, `voteCount`, `alive`) our voting system will use to manage Candidate state. In the constructor we set our default for whenever new Candidate is created.

In addition, we have declared one class property inside the constructor argument list. This is a shorthand to allow that argument to be passed in when object is created, set that properties value.

There is a basic Vote class there as well but nothing different nor important is there but it could be extended with additional information, e.g. when vote was made.

## CallResponse class

When creating backend systems you always want to have a robust and consistent way you can get data back to be manipulated or displayed on frontend so we define this basic structure to be used in all out call functions.

```
@nearBindgen
class CallResponse {
	constructor(
		public success: boolean,
		public messages: string[],
	) {

	}
}
```

`success` property indicates if request was successful to know how and which response to display to user in a user friendly way.
`messages` in our case explain what went wrong with out request to end user or communicate other information - in our case for entertainment :)

## ActionLog class

All important system should have an ability to track down who changed what in the system. In our case it's who (which user) interacted with our system in what way (voted, added candidate, started a new election).

```
@nearBindgen
class ActionLog {
	constructor(
		public user: string,
		public action: string,
	) {

	}
}
```

`user` contains their username and `action` explains what they did.

## Data Storage

In order for system to work and not just forget what it knows after each command is executed we need to preserve that data somewhere. You can read more about `PersistentUnorderedMap` and other storage options here - https://docs.near.org/docs/concepts/data-storage#persistentunorderedmap

```
const candidates = new PersistentUnorderedMap<number, Candidate>("m");
const votes = new PersistentUnorderedMap<string, Vote>("n");
const logs = new PersistentUnorderedMap<number, ActionLog>("b");
```

Important thing to keep in mind is that you need to specify different name (`m`, `n`, `b`) for each one of them. Otherwise, they all will point to the same data causing unexpected results.

## Helpers

In order to keep the rest of the code cleaner it's often useful to create some helper functions that handle repeated tasks for you.

### Random Number & Boolean Generation

In NEAR to work with randomness we need to user `math.randomBuffer` that comes from `near-sdk-core` - https://near.github.io/near-sdk-as/globals.html that returns X amount of random numbers for you to work with.

```
function randomNumber(min: number = 0, max: number = 100): i32 {
	const buf = math.randomBuffer(4);
	return i32(min + (((((0xff & buf[0]) << 24) |
	((0xff & buf[1]) << 16) |
	((0xff & buf[2]) << 8) |
	((0xff & buf[3]) << 0)) as number) % (max - min)
	));
}

function randomBoolean(): boolean {
	return randomNumber(0, 100) >= 50;
}
```

To get random boolean value we reuse our existing helper function `randomNumber` for simplicity.

### Responses & Logs

Other common functions I tend to use is `response` and `log`. These wrappers are very useful in a case where at some point you want to change how they work then you only have to do it in one place instead of digging through the whole code.

```
function response(messages: string[], success: boolean): CallResponse {
	return new CallResponse(success, messages)
}

function log(message: string): void {
	const logEntries = logs.keys();
	logs.set(logEntries.length, new ActionLog(
		context.sender,
		message,
	));
}
```

# Add Candidate

First, we need ability to add new candidates to the election.

```
export function addCandidate(name: string): CallResponse {
	const candidate = new Candidate(name);

	candidates.set(candidates.length, candidate);

	log('Added candidate ' + candidate.name);

	return response([candidate.name + ' successfully added to candidate list!'], true);
}
```

There is only one argument `name` that we require and that's used to initialize a new Candidate. After that we store this candidate in our storage using `candidates.set(candidates.length, candidate)`. First parameter for set method is the key on how we want to identify and later retrieve our candidate. I simply use `candidates.length` which returns number of already existing candidates giving us a unique identifier for our Candidate.

After that we log user action and respond with user friendly message.

# View Candidate

Users will want to know all available candidates and their votes. Thanks to `PersistentUnorderedMap` we can very easily return all our candidates using `candidates.entries()` that creates an array of key/value pairs of our candidates.

```
export function viewCandidates(): MapEntry<number, Candidate>[] {
	return candidates.entries();
}
```

# Vote

And now, the most important function - ability to vote! This one is a bit more complex so I'll split it in parts.

```
export function vote(candidateId: string): CallResponse {
	const candidateIntId = parseInt(candidateId);

	if (votes.contains(context.sen
	der)) {
		return response(['You have already voted!'], false);
	}
```

First, we check if user has already made a vote in the past in current election. If that's the case we respond with error message and mark request as unsuccessful.

```
	const candidate = candidates.get(candidateIntId);

	if (candidate == null) {
		return response(["Candidate doesn't exist!"], false);
	}
```

Then we check if candidate user is voting for exists in the system. If it doesn't then we again respond with error message.

```
	if (candidate.alive) {
		candidate.voteCount += 1;

		candidates.set(candidateIntId, candidate);

		votes.set(context.sender, new Vote(candidateIntId));

		log('Voted for ' + candidate.name);

		return response(["Successfully voted for " + candidate.name + "!"], true);
```

Now, that we know user can vote for a valid candidate, we check if candidate is alive. If he is then we increase candidates `voteCount`, save the updated candidate and store information that this user voted for this participant.

After that we log their action and respond with successful request and message.

```
	} else {
		return response([
			"You can't vote for the dead!",
			"Or can you?",
			"No, no, you can't :p",
		], false);
	}
}
```

Finally, if candidate is indeed dead (can happen through other commands in the system) we return a small entertaining joke message and mark request as failed.

# That's It Folks!

This covers the basic functionallity of our voting system. If you keep scrolling down in `assembly\main.ts` you will find additional commands to explore and experiment with.

Now that we have gone through the code - we want to build and deploy our voting system to NEAR.

# Build & Deploy

Before you build and deploy your Smart Contract (our Voting System) make sure that you have logged in using NEAR by typing `near login` in console and updating `CONTRACT_NAME` in `src\config.js` with your username as that's where the contract will be deployed.

Now, build the contract by executing command below in root directory of the project.

```
yarn run build:contract
```

Then deploy it using command below

```
near deploy
```

# Commands

Now, that your Smart Contract is deployed you can call these methods to interact with it.

**IMPORTANT** - replace `near-challenge-7.testnet` and `martint.testnet` with your own username.

## Add Candidate

**Command**

```
near call near-challenge-7.testnet addCandidate '{ "name": "Trump" }' --accountId martint.testnet --gas 300000000000000
```

**Result**

```
{
  success: true,
  messages: [ 'Trump successfully added to candidate list!' ]
}
```

## View Candidates

**Command**

```
near view near-challenge-7.testnet viewCandidates
```

**Result**

```
[
  {
    key: 0,
    value: { avatar: '72724247', voteCount: 0, alive: true, name: 'Trump' }
  },
  {
    key: 1,
    value: { avatar: '72724317', voteCount: 3998, alive: true, name: 'Trump Junior' }
  },
  {
    key: 2,
    value: { avatar: '72724344', voteCount: 1, alive: true, name: 'Martin' }
  }
]
```

## Vote

**Command**

```
near call near-challenge-7.testnet vote '{ "candidateId": "2" }' --accountId martint.testnet --gas 300000000000000
```

**Result**

```
{
  success: true,
  messages: [ 'Successfully voted for Martin!' ]
}

or

{
  success: false,
  messages: [ 'You have already voted!' ]
}
```

Feel free to go through the rest of the commands/functions in `assembly\main.ts` and see exactly what they do and how they respond here - https://github.com/MartinTale/near-challenge-7/blob/main/DETAILS.md
