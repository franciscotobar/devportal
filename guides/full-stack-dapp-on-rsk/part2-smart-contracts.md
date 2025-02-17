---
layout: rsk
title: The Complete Full Stack dApp Guide on RSK - Part 2:Smart Contracts
description: 'Build a complete full stack decentralized application on RSK, from front-end to smart contracts'
tags: the-complete-full-stack-dapp-guide, full-stack, dapp, tutorial, overview, front-end, guides, smart-contracts, web3, bitcoin, rsk, peer-to-peer, dapp-examples, blockchain
---

This is the second part of the series
on [building a **complete full stack dApp on RSK**](/guides/full-stack-dapp-on-rsk/).

In this article, we will cover installation and configuration
of the development environment for coding, compiling and testing smart contracts.
You’ll also learn how to connect to the RSK network,
and deploy your smart contracts on it.

![The Complete Full Stack dApp guide](/assets/img/guides/complete-full-stack-dapp/Fullstack-tutorial2.jpg)

We want to build a **simple voting application**.
RSK allows us to build decentralised applications,
and solidity is the smart contract language we would be using alongside the front end.

Before we get started, please read the
[The Complete Full Stack dApp Guide on RSK - Part 1: Overview](/guides/full-stack-dapp-on-rsk/part1-overview/)

## Prerequisites

Prior to commencing this tutorial, please ensure that you have installed the following RSK workshop pre-requisites on your system:

- [POSIX compliant shell](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#posix-compliant-shell)
- [NodeJs](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#nodejs)
- [Truffle](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#truffle)
- [curl](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#curl)
- [Code editor](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#code-editor)
- [Mnemonics](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#mnemonics)
- [Java](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#java)
- [RSKj](https://github.com/bguiz/workshop-rsk-prereqs/blob/master/walkthru.md#rskj)

## 1. Initialise the project

Use `git` to make a copy of this repo, and use `npm` to install dependencies.

```shell
git clone git@github.com:bguiz/workshop-rsk-full-stack-dapp.git
cd workshop-rsk-full-stack-dapp
npm install
```

### 1.1 Setup additional project files

Enter the following commands into the terminal.

Get the current gas price for both RSK Testnet and RSK Mainnet:

```shell
curl https://public-node.testnet.rsk.co/ -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' > .testnet.gas-price.json
curl https://public-node.rsk.co/ -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' > .mainnet.gas-price.json
```

Generate a [Mnemonic](https://en.bitcoinwiki.org/wiki/Mnemonic_phrase) phrase:

```shell
mnemonics > .testnet.seed-phrase
```

> Ensure RSKj (Regtest) is running in a separate terminal.
> To setup Regtest see the prerequisites section above

Then open up this directory in your code editor.

Observe that we have the following files:

- `truffle-config.js`:
  Truffle has already been pre-configured to connect to your choice of
  RSK Regtest, RSK Testnet, or RSK Mainnet.
- `contracts/Migrations.sol` and `migrations/1_initial_migration.js`:
  These are auto-generated by Truffle projects,
  and has to do with deployments, and we do not need to care about them.
- `contracts/Election.sol` and `migrations/2_election_migration.js`:
  These are a smart contract and its corresponding deployment script.
  The solidity file: `.sol` is the implementation, and we'll follow the steps below to set up the voting contract.
- `test/Election.spec.js`
  This is the test specification.

### 1.1.2 Start a blockchain locally

Enter the following command in the terminal to start the truffle console.

```shell
truffle develop
```

You can see a list of available accounts below that are already created and ready to use; you can also see the port (`9545`) where the blockchain is running. This is an in-memory blockchain, which means that **no other peers** are going to connect and interact with it automatically. It’s only for **testing** purposes.

Output:
```shell
owanate@MacBook-Pro workshop-rsk-full-stack-dapp % truffle develop
Truffle Develop started at http://127.0.0.1:9545/

Accounts:
(0) 0xe265436fb52323a40cc4cdff9ca57d2d27bc8a25
(1) 0x9faf588e44a32b3b8028832f142445f6f436f570
(2) 0xd936c4f489bfc4c14c69281c69dd3416f8d1978e
(3) 0xb6b7ec811b3d1cca08df7404569eb0f808159f37
(4) 0xc451e513b81b3b4fc627a8e0378b12e4160522c7
(5) 0x03f00de5c3281f8c995e031f3e72ca7b9ced4111
(6) 0xf07ae6cf2450cc46a7602186b5b8cedcc441ca9e
(7) 0xcb31275bbeaa4a36f1106e2fef05e45e12d21ad8
(8) 0x5d9bbd729c1a70ad9bc9be55fed1d21ea7d5227c
(9) 0x2cf02625b40c39d1fa3df5e92a0b8809977ac593

Private Keys:
(0) 88e8dd5b9100d8e9f595809a251dd91b82f66924ec377b0580ebe0779ce6c5ae
(1) 53e7390a65b68e122f37b3b3de89fa21a88023d733ec8f6a8779ed02cfff1c82
(2) a37d67410ae97aac8d08467024b59f623dbd653b2d6eb7598ed1b3f26a5c3de7
(3) 6d481af222ae78fddd94ab8bc72118eea05321a863925987d715a62c459423cb
(4) 431975ed3f2f4d224b5c6085a6689cc1f6e135f36f7b76c72b31af3990a324f1
(5) abf11a0083a7f3153e800a324402ff4e4488d5660d5b2df9e4edc006398d61ca
(6) bf0b2865ad4bd8dd985e3e2ce0d46766f7ae64ad2c40991de1de052c882a6b94
(7) a40925e7bfdc6d2e70721152eae1b507a7c9299a2a7ef25152e082b8d6351713
(8) 3c0c735fc0b20a229611269a701cc4269fe7f7aa9fd1a036ea44cf7f3e2452a7
(9) 546a5da5be8996b4e45a331d73b6cc1b5498ba908a8a0c676ad3f983744eeed9

Mnemonic: outdoor organ farm keep pitch remember click crack oppose scare scout erode

⚠️  Important ⚠️  : This mnemonic was created for you by Truffle. It is not secure.
Ensure you do not use it on production blockchains, or else you risk losing funds.
```

## 2. Creating the Smart contract

The initial set up code for the dApp can be viewed in the [diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/30ab50b.diff). We'll implement the functions in the smart contract as we go along.

To locate the smart contract, go to the contract folder and open the file called `Election.sol` in your code editor.

![RSK Full Stack dApp - Election.sol](/assets/img/guides/complete-full-stack-dapp/Election.png/)

> Note that there are several places marked with `// TODO` as comments. This tutorial will guide you on how to complete this implementation.

### 2.1. Code Walkthrough

```solidity
pragma solidity ^0.5.0;

contract Election {
    // Model a Candidate
    struct Candidate {
        uint id;
        string name;
        uint voteCount;
    }

    // Store accounts that have voted
    mapping(address => bool) public voters;
    // Read/write candidates
    mapping(uint => Candidate) public candidates;
    // Store Candidates Count
    uint public candidatesCount;

    constructor () public {
        // TODO set default candidates
    }

    function addCandidate (
        string memory _name
    ) private {
        // TODO store new candidate in state variables
    }

    function vote (
        uint _candidateId
    ) public {
        // TODO check that account hasn't voted before
        // and that candidate is valid
        require(
            _candidateId > 0 &&
            _candidateId <= candidatesCount);

        // TODO record that voter has voted
    }
}
```

The first line - `pragma` -  declares the version of Solidity you wish to write your code in.
This is first done in all solidity smart contracts.
The declaration of the smart contract starts with the keyword `contract`,
then the name of the contract.
In this instance `Election` is the name of our smart contract.

Next, we declared a candidate struct, which will hold three variables:
`id`, `name`, and `voteCount`.
[Structs](https://solidity.readthedocs.io/en/v0.7.1/structure-of-a-contract.html?highlight=struct#struct-types)
in Solidity are custom defined types that can group several variables,
similar to how variables are grouped in [Object Oriented Programming Languages](https://en.wikipedia.org/wiki/Object-oriented_programming).

- The lines `uint id` and `uint voteCount` declare a state variable called `id` and `voteCount`, both of the type `uint` (unsigned integer with 256 bits).
- The line `string name` declares a state variable called `name` of type `string`.

Next, we store the accounts that have voted
by `mapping` their `address` to type `bool`,
and setting its accessibility to public using the keyword `public`.
Read more on [Variable Declarations in Solidity](https://solidity.readthedocs.io/en/v0.7.1/style-guide.html?highlight=bool#mappings).

The `constructor` function gets called when the smart contract is deployed to the blockchain.
The lines `addCandidate (Carrot) and addCandidate (Potato)`
in the constructor would be initialised upon contract creation.
Read about [Constructors in Solidity](https://solidity.readthedocs.io/en/v0.7.1/contracts.html#constructor).

The function `addCandidate` declares a candidate to the `_name` variable
with type `string`, sets it to `private` -
eaning this function can only be called within this contract.
Read about [Function Visibility Specifiers](https://solidity.readthedocs.io/en/v0.7.1/cheatsheet.html?highlight=private%20function#function-visibility-specifiers) in Solidity.

The function `vote` declares a `_candidateId` of type `uint`,
and sets the visibility of the function to `public` -
this means the function is visible both externally and internally.

### 2.2.  Implementing the `constructor` and `addCandidate` function

Let's implement the `constructor` and `addCandidate` function.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/61345bc.diff).

Let's begin with the `addCandidate` function:

```solidity
    function addCandidate (
        string memory _name
    ) private {
        candidatesCount++;
        candidates[candidatesCount] =
            Candidate(candidatesCount, _name, 0);
    }
```

Each time this function is called,
it increments the number of candidates,
and stores the candidate data inside a mapping.

```solidity
    constructor () public {
        addCandidate("Carrot");
        addCandidate("Potato");
    }
```

For the constructor, we simply invoke
the `addCandidate` function twice,
with hard coded values.
In this election, voters get to pick between
a carrot and a potato. 😛

### 2.3. Implementing the `vote` Function

Let's implement the `vote` function.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/0b031b73d6f48689cb7c6d5f183ab8b9c55307a9).
It should now look like this:

```solidity
function vote (
        uint _candidateId
    ) public {
        // TODO check that account hasn't voted before
        require(
            !voters[msg.sender]);
        // require a valid candidate
        require(
            _candidateId > 0 &&
            _candidateId <= candidatesCount);

        // TODO record that voter has voted
        voters[msg.sender] = true;

        // update candidate vote Count
        candidates[_candidateId].voteCount++;
    }
```

The `vote` function also checks if the present caller of the contract
(the candidate) has voted before,
if not it proceeds to check if the candidate is valid.
Then it records the candidate's vote and assigns the candidate (address)
to the voters array and updates the votes and candidate's count.

### 2.4. The Updated Smart Contract

Find the full code in the `Election.sol` file below or in the [Full stack dApp repo](https://github.com/bguiz/workshop-rsk-full-stack-dapp/blob/feat/complete-a/contracts/Election.sol)

```solidity
pragma solidity ^0.5.0;

contract Election {
    // Model a Candidate
    struct Candidate {
        uint id;
        string name;
        uint voteCount;
    }

    // Store accounts that have voted
    mapping(address => bool) public voters;
    // Read/write candidates
    mapping(uint => Candidate) public candidates;
    // Store Candidates Count
    uint public candidatesCount;

    constructor () public {
        addCandidate("Carrot");
        addCandidate("Potato");
    }

    function addCandidate (
        string memory _name
    ) private {
        candidatesCount++;
        candidates[candidatesCount] =
            Candidate(candidatesCount, _name, 0);
    }

    function vote (
        uint _candidateId
    ) public {
        // require that they haven't voted before
        require(
            !voters[msg.sender]);

        // require a valid candidate
        require(
            _candidateId > 0 &&
            _candidateId <= candidatesCount);

        // record that voter has voted
        voters[msg.sender] = true;

        // update candidate vote Count
        candidates[_candidateId].voteCount++;
    }
}
```

### 2.5. Compiling the Smart Contract

To compile the contracts, in the Truffle console, run this command:

```shell
truffle(develop)> compile
```

Output:

```shell
Compiling your contracts...
===========================
> Compiling ./contracts/Election.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /Users/owanate/Documents/Projects/TutorialPractice/workshop-rsk-full-stack-dapp/build/contracts
> Compiled successfully using:
   - solc: 0.5.7+commit.6da8b019.Emscripten.clang
```

## 3. Testing the Smart Contract

Tests are extremely important when dealing with systems of trust or high value.
Tests can be written in either Solidity or with Javascript
(using Truffle’s abstractions).
In this tutorial, we'll use Javascript.

The Truffle framework embeds support for the popular **Mocha** test framework for Javascript.
Here’s what some basic tests look like for the above contract,
located in the `test` folder,
you'll find a file with the name `Election.spec.js`:

![RSK Full Stack dApp - Test](/assets/img/guides/complete-full-stack-dapp/Election.spec.png)

### 3.1. Test File

The initial code for testing the smart contract is copied below.
We'll implement the specifications as we go along.

```javascript
const assert = require('assert');

const Election = artifacts.require('Election');

const BN = web3.utils.BN;

contract('Election', function(accounts) {
  let electionInstance;

  before(async () => {
    electionInstance = await Election.deployed();
  });

  it('initializes with two candidates', async () => {
    // TODO specification code
  });

  it('it initializes the candidates with the correct values', async () => {
    // TODO specification code
  });

  it('disallows voting on invalid candidates', async () => {
    // TODO specification code
  });

  it('disallows double voting', async () => {
    // TODO specification code
  });

  it('allows a voter to cast a vote', async () => {
    // TODO specification code
  });
});

```

### 3.2. Testing the initial state of candidate count

Let's test if our function does initialize with two candidates.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/bbed126.diff).
It should now look like this:

```javascript
  it('initializes with two candidates', async () => {
    const count = await electionInstance.candidatesCount();
    assert.strictEqual(
      count.toString(), '2');
  });
```

> This simply checks that the number of candidates is equal to 2.

### 3.3. Testing the initial values of candidate data

Let's test if our function would initialize the candidates with the correct values.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/7addd2c.diff).
It should now look like this:

```javascript
it('it initializes the candidates with the correct values', async () => {
    const candidate1 = await electionInstance.candidates(1);
    assert.strictEqual(
      candidate1.id.toString(), '1',
      'contains the correct id');
    assert.strictEqual(
      candidate1.name, 'Carrot',
      'contains the correct name');
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'contains the correct votes count');
    const candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate2.id.toString(), '2',
      'contains the correct id');
    assert.strictEqual(
      candidate2.name, 'Potato',
      'contains the correct name');
    assert.strictEqual(
      candidate2.voteCount.toString(), '0',
      'contains the correct votes count');
  });

```

> This retrieves each of the candidates,
> and checks that they have the names that we used in our constructor,
> and that they have zero votes to begin with.

### 3.4. Testing for disallow voting on invalid candidates

Let's test if our function will disallow voting on invalid candidates.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/3bf8bee.diff).
It should now looks like this:

```javascript
it('disallows voting on invalid candidates', async () => {
    let err;
    try {
      await electionInstance.vote(1234, { from: accounts[3] });
    } catch (ex) {
      err = ex;
    }
    assert(err, 'expected transaction to revert');
    assert(err.message.indexOf('revert') >= 0,
      'error message must contain revert');
    const candidate1 = await electionInstance.candidates(1);
    const candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '0',
      'candidate 2 did not receive any votes');
  });

```

> This test case is for a "failure path",
> that is testing that we don't allow a user
> to do something that they should not be allowed to do.
> In this case, that they cannot vote for a candidate who does not exist.

### 3.5. Testing for disallow double voting

Let's test if our function will disallow double voting.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/2dc5fda.diff).
It should now look like this:

```javascript
it('disallows double voting', async () => {
    const candidateId = 2;
    let err;
    let candidate1;
    let candidate2;

    try {
      await electionInstance.vote(candidateId, { from: accounts[2] });
    } catch (ex) {
      err = ex;
    }
    assert(!err, 'expected transaction not to revert');
    candidate1 = await electionInstance.candidates(1);
    candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '1',
      'candidate 2 did receive a vote');

    try {
      await electionInstance.vote(candidateId, { from: accounts[2] });
    } catch (ex) {
      err = ex;
    }
    assert(err, 'expected transaction to revert');
    assert(err.message.indexOf('revert') >= 0,
      'error message must contain revert');
    candidate1 = await electionInstance.candidates(1);
    candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any extra votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '1',
      'candidate 2 did not receive any extra votes');
  });

```

> This test case is for a different "failure path".
> In this case, that a voter cannot vote additional times
> after they have already voted once.

### 3.6. Testing for allow a voter to cast a vote

Let's test if our function will allow a voter to cast a valid vote.
[Diff file for change](https://github.com/bguiz/workshop-rsk-full-stack-dapp/commit/57e39e5.diff).
It should now look like this:

```javascript
it('allows a voter to cast a vote', async () => {
    const candidateId = new BN(1);
    await electionInstance
      .vote(candidateId, { from: accounts[0] });
    const voted = await electionInstance.voters(accounts[0]);
    const candidate = await electionInstance.candidates(candidateId);
    assert(voted, 'the voter was marked as voted');
    assert.strictEqual(
      candidate.voteCount.toString(), '1',
      'increments the candidate\'s vote count');
  });
```

### 3.7. The Updated Test Code

Find the full test code in the `Election.spec.js` file below or in
the [Full stack dApp repo](https://github.com/bguiz/workshop-rsk-full-stack-dapp/blob/feat/complete-a/test/Election.spec.js)


```javascript
const assert = require('assert');

const Election = artifacts.require('Election');

const BN = web3.utils.BN;

contract('Election', function(accounts) {
  let electionInstance;

  before(async () => {
    electionInstance = await Election.deployed();
  });

  it('initializes with two candidates', async () => {
    const count = await electionInstance.candidatesCount();
    assert.strictEqual(
      count.toString(), '2');
  });

  it('it initializes the candidates with the correct values', async () => {
    const candidate1 = await electionInstance.candidates(1);
    assert.strictEqual(
      candidate1.id.toString(), '1',
      'contains the correct id');
    assert.strictEqual(
      candidate1.name, 'Carrot',
      'contains the correct name');
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'contains the correct votes count');
    const candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate2.id.toString(), '2',
      'contains the correct id');
    assert.strictEqual(
      candidate2.name, 'Potato',
      'contains the correct name');
    assert.strictEqual(
      candidate2.voteCount.toString(), '0',
      'contains the correct votes count');
  });

  it('disallows voting on invalid candidates', async () => {
    let err;
    try {
      await electionInstance.vote(1234, { from: accounts[3] });
    } catch (ex) {
      err = ex;
    }
    assert(err, 'expected transaction to revert');
    assert(err.message.indexOf('revert') >= 0,
      'error message must contain revert');
    const candidate1 = await electionInstance.candidates(1);
    const candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '0',
      'candidate 2 did not receive any votes');
  });

  it('disallows double voting', async () => {
    const candidateId = 2;
    let err;
    let candidate1;
    let candidate2;

    try {
      await electionInstance.vote(candidateId, { from: accounts[2] });
    } catch (ex) {
      err = ex;
    }
    assert(!err, 'expected transaction not to revert');
    candidate1 = await electionInstance.candidates(1);
    candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '1',
      'candidate 2 did receive a vote');

    try {
      await electionInstance.vote(candidateId, { from: accounts[2] });
    } catch (ex) {
      err = ex;
    }
    assert(err, 'expected transaction to revert');
    assert(err.message.indexOf('revert') >= 0,
      'error message must contain revert');
    candidate1 = await electionInstance.candidates(1);
    candidate2 = await electionInstance.candidates(2);
    assert.strictEqual(
      candidate1.voteCount.toString(), '0',
      'candidate 1 did not receive any extra votes');
    assert.strictEqual(
      candidate2.voteCount.toString(), '1',
      'candidate 2 did not receive any extra votes');
  });

  it('allows a voter to cast a vote', async () => {
    const candidateId = new BN(1);
    await electionInstance
      .vote(candidateId, { from: accounts[0] });
    const voted = await electionInstance.voters(accounts[0]);
    const candidate = await electionInstance.candidates(candidateId);
    assert(voted, 'the voter was marked as voted');
    assert.strictEqual(
      candidate.voteCount.toString(), '1',
      'increments the candidate\'s vote count');
  });
});
```

Finally, we have a "happy path" test case,
where we check that a user is allowed to vote
when they are following the rules set out in the smart contract.

## 4. Running the Tests

Open a new terminal in the project directory.

Enter the command below into the terminal:

```shell
npm run test
```

Output:

```javascript
$ npm run test
> truffle test --network regtest

Gas price Testnet: 65000004
Using network 'regtest'.


Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



  Contract: Election
    ✓ initializes with two candidates
    ✓ it initializes the candidates with the correct values
    ✓ disallows voting on invalid candidates (2121ms)
    ✓ disallows double voting (4107ms)
    ✓ allows a voter to cast a vote (2064ms)


  5 passing (8s)
```

Hurray! You can see all our tests passed!!

> Note: We are using the `regtest` network as specified
> in `scripts.test` in `package.json` to test our smart contract.

## 5. Deploying the Smart Contract on RSK

Remember that we spun up `localhost` blockchain earlier?
Well that isn't going to be very useful for smart contracts,
because a single person (you) can destroy
all the blockchain data, which includes smart contract data too.
Not to mention, there were no peers, and therefore
no [consensus](https://news.bitcoin.com/a-history-of-blockchain-consensus-mechanisms/).

We want something that **is not** ephemeral,
and **is** verified by a network of peers.
So let's deploy this smart contract to a **real blockchain**.
RSK offers two public networks: Testnet and Mainnet.
As the name implies, Testnet is for testing purposes.
This is useful for development, as we're doing now.

### 5.1 Get an RSK Wallet

To get a wallet, we first need to get a **mnemonic** code.
We are going to use this web app:
[iancoleman.io/bip39](https://iancoleman.io/bip39/).
This may not be used for any ‘real’ wallet;
it’s not a secure way to generate a private key!
We are going to use it just for learning the basics.

In the ‘Generate a random mnemonic’ field, we select '12 words' and generate it.
Then tap on ‘Generate’.
The result appears in the BIP39 Mnemonic field.
They should be 12 random words like the words in the image:

![RSK full stack dApp - Mnemonic](/assets/img/guides/complete-full-stack-dapp/Mnemonic.png)

In the terminal, inside the project root folder,
a text file named `.testnet.seed-phrase` has already been generated
during the additional project files setup above.

Do you remember your mnemonic?

### 5.2 Connect Truffle to RSK public network

To connect to RSK, we are going to modify the Truffle configuration.
We are going to use a provider that allows us to connect to any network but unlocking an account locally.
We are going to use `@truffle/hdwallet-provider@1.0.35`
and Node.Js >= `12.18.2`.
The `@truffle/hdwallet-provider@1.0.35` has already been installed for you
when you used the `npm install` command

### 5.3 Configure `truffle-config.js` file

Read about how to [Configure Truffle to Connect to RSK](/kb/configure-truffle-to-rsk) using Truffle's default configuration values (based on Ethereum), and using 2 relatively new config options to configure Truffle to better connect to an RSK node.

In the root folder directory, locate the `truffle-config.js` file.
Truffle has already been pre-configured to connect to
your choice of RSK Regtest or RSK Testnet.

The `hdwallet-provider` allows us to connect to any network by unlocking an account locally, including the RSK networks.
For example, to connect to the RSK Testnet,
we use the mnemonic stored in the text file
`.testnet.seed-phrase`.

> A key part to look out for in the code above is the `network` section, under `testnet`. This tells truffle how it should connect to the RSK Testnet:

```javascript
networks: {
    testnet: {
      provider: () => new HDWalletProvider(
        testnetSeedPhrase,
        'https://public-node.testnet.rsk.co/',
      ),
      // Ref: http://developers.rsk.co/rsk/architecture/account-based/#chainid
      network_id: 31,
      gasPrice: Math.floor(gasPriceMainnet * TESTNET_GAS_MULT),
      networkCheckTimeout: 1e6,
    },
 }
```

The above:

- Configures a connection to the RSK Testnet
- Uses `HDWalletProvider` to generate a set of
  wallets from the mnemonic, according to
  [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- Specifies a network ID for 31
- Specifies that we should use 110% of the minimum gas price in transactions

### 5.4 Get the current gas price of Testnet

Whenever one submits a transaction to a blockchain network,
and that transaction is added to the blockchain,
a fee must be paid.
This fee is known as gas, and is a measure of the amount of
computational and storage costs consumed by that transaction.
Since we're deploying a smart contract and interacting with it,
we will need gas to do so.

The price of gas varies over time, so let's
update the current gas price of the Testnet network,
and save it to `.testnet.gas-price.json`,
using the following command:

```shell
curl https://public-node.testnet.rsk.co/ -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' > .testnet.gas-price.json
```

> A file named `.testnet.gas-price.json` previously generated will be updated if there are any changes in gas price. You can locate the file at the root folder.

This result below shows the output to the file when the code above is entered into the terminal. Open `.testnet.gas-price.json` to view the changes to the file.

```json
{"jsonrpc":"2.0","id":1,"result":"0x3dfd244"}
```

The `result` value is presented in hexadecimal.

> You can connect to both Regtest and Testnet, using the configurations in the `truffle-config.js` file.
> However, in this guide, we'll focus on Testnet.

### 5.5 Connecting to the RSK Testnet

To connect to the RSK Testnet, open another terminal in the same folder directory, enter the following command below into the terminal.

```shell
truffle console --network testnet

# Gas price Testnet: 65000004
```

### 5.6. Get test funds on RSK Testnet

To find out which address was unlocked using the mnemonic, enter the command below into the terminal:

```shell
truffle(testnet)> Object.keys(web3.currentProvider.wallets)
```

Output:

```shell
truffle(testnet)> Object.keys(web3.currentProvider.wallets)
[
  '0x460a9914c73efc0aacc39e011b239158e15bca1c',
  '0x263de679f78ee334ac34e905548d8b57cca1fadd',
  '0x3e913ee2b5bbc2c3143528f3e7942e49954d00cc',
  '0xb4604ed7161904e72cea49ee30c0453797ec3de4',
  '0xfe4777644121f0aeaf2f09afa4c1d95e4dbce8e2',
  '0xf9d9cc0f0977a76ab84164106cd6b61a6706d816',
  '0x7ca3b938932dd412e92074538c74a4b183dcb3bb',
  '0x7748d18056e11086a18c82a8b18d706865c9a3ae',
  '0x8938ae9b5e8f0d52163ab09376c5106e228eacd2',
  '0x8c4b878d03e97d9c60b69241ce44192cac714fbf'
]
```

> Let's save the address of the first account in a variable named `account`
> within the console. We will be using it soon.

Enter the following code into the terminal.

```shell
truffle(testnet)> var account = Object.keys(web3.currentProvider.wallets)[0]
# undefined
```

Then enter the command below;

```shell
truffle(testnet)> account
# '0x460a9914c73efc0aacc39e011b239158e15bca1c'
```

> RSK Testnet is a free network.
> You still need to pay gas fees to interact with it,
> but you can obtain that for free in a faucet.
> Let’s go to the RSK Faucet: [faucet.testnet.rsk.co](https://faucet.testnet.rsk.co/).

![RSK faucet - 1](/assets/img/guides/complete-full-stack-dapp/TestFunds1.png)

> Steps:
>
> 1. Enter the address of the account obtained earlier
> 2. Enter the "captcha" code
> 3. Submit form
> 4. Wait for the transaction to complete

> Congratulations, you’ve received a transaction from the faucet.

![RSK faucet - 2](/assets/img/guides/complete-full-stack-dapp/TestFunds2.png)

> Now, let’s check our balance in the terminal, enter the command below:

```
truffle(testnet)> web3.eth.getBalance(account, (err, res) => console.log(res))
# 50000000000000000
```

### 5.7. Deploy the Contract to RSK Testnet

To deploy the contracts, we are going to follow the same steps we made in our local network, but this time there will be a little delay because we are publishing the contracts to a public network!

Enter the following command below into the terminal:

```shell
truffle(testnet)> compile
```

Output:

```shell
Compiling your contracts...
===========================
> Compiling ./contracts/Election.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /Users/owanate/Documents/Projects/TutorialPractice/workshop-rsk-full-stack-dapp/build/contracts
> Compiled successfully using:
   - solc: 0.7.0+commit.9e61f92b.Emscripten.clang
```

> Note: Please make sure the build folder is clean. else, the following will be the output and the contract will not be compiled:

```shell
Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.
```

If everything is working properly,
you should be able to deploy to multiple networks
without cleaning out the build folder.

After the contract is successfully compiled,
enter the command below into the terminal.

```shell
truffle(testnet)> migrate --network testnet
```

Output:

```shell
truffle(testnet)> migrate --network testnet

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'testnet'
> Network id:      31
> Block gas limit: 6800000 (0x67c280)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0x07a1ba12945c8a623071613b4f15a57e84ed6060eadf8d86dfd37c81050105b9
   > Blocks: 1            Seconds: 57
   > contract address:    0xCC6838AeF5CFBd1f70C853Af1fAB71BBC808fCfc
   > block number:        1238308
   > block timestamp:     1602096652
   > account:             0x460A9914c73EFc0aaCc39E011B239158e15BCa1c
   > balance:             0.049936826314465808
   > gas used:            182191 (0x2c7af)
   > gas price:           0.071500004 gwei
   > value sent:          0 ETH
   > total cost:          0.000013026657228764 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.000013026657228764 ETH


2_election_migration.js
=======================

   Deploying 'Election'
   --------------------
   > transaction hash:    0xd6927752c8256416f891740e4e292a8626b3200679bd1b0a893f66b19f60af6c
   > Blocks: 1            Seconds: 53
   > contract address:    0x6bF38368521C7bC92F0DFa1a17bE7F4B77EE527b
   > block number:        1238311
   > block timestamp:     1602096765
   > account:             0x460A9914c73EFc0aaCc39E011B239158e15BCa1c
   > balance:             0.049901636300497136
   > gas used:            450170 (0x6de7a)
   > gas price:           0.071500004 gwei
   > value sent:          0 ETH
   > total cost:          0.00003218715680068 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:     0.00003218715680068 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.000045213814029444 ETH
```

> Once the contract is deployed we can use the `deployed()` method
> as we did in the local blockchain.
> To see the interaction with the contract we can access it via
> [RSK Testnet explorer](https://explorer.testnet.rsk.co/)
> and search using the contract address, transaction hash or block number.
> All interactions with our contract will appear in the explorer!

Congratulations for getting this far!
Now you're a smart contract developer😉😉. View the entire code for the
[Complete Full Stack dApp repo](https://github.com/bguiz/workshop-rsk-full-stack-dapp/tree/feat/complete-a)

**Next:** Now let's build out the front end!
Check out the third part of
[The Complete Full Stack dApp Guide on RSK Part 3](/guides/full-stack-dapp-on-rsk/part3-front-end/)

## Further Reading
- [Solidity Documentation](https://solidity.readthedocs.io/en/v0.7.1/index.html)
- [Mocking Solidity Smart Contracts](https://dappsdev.org/hands-on/testing/solidity-mocks/)
- [Deploy A Smart Contract](https://developers.rsk.co/tutorials/deploy-smart-contracts/)

----

[RSK Developers Portal](https://github.com/rsksmart/devportal) |
[Contact us on our community Slack](/slack/) |
[Visit our Webinars Section](https://developers.rsk.co/webinars/)
