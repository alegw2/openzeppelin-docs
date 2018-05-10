---
id: test-environment
title: Setting up your test environment
---

To start developing Ethereum apps (or DApps, for decentralized applications),
you’ll need a client to connect to the network.
It will act as your window to the distributed network, and provide a view of
the blockchain, where all the EVM state is represented.

There are various compatible clients for the protocol, At OpenZeppelin, we've found
[testrpc](https://github.com/ethereumjs/testrpc) to be the most
developer-friendly option (rather than the slightly more popular `geth`).
Install it and run it – you may need to prepend `sudo` depending on your setup:

```sh
npm install -g ethereumjs-testrpc
testrpc
```

You should run testrpc in a new terminal and leave it running while you
develop. Each time you run testrpc, it will generate 10 new addresses with
simulated test funds for you to use. This is not real money and you’re safe to
try anything with no risk of losing funds. `testrp` state is volatile – every time you close it, 
the state of your node and accounts will be cleared.

The most popular language for writing smart contracts in Ethereum is
[Solidity](https://solidity.readthedocs.io/en/latest/), so we’ll be using that.
We’re also using the [Truffle development framework](https://github.com/trufflesuite/truffle), which helps with 
smart contract creation, compiling, deployment and testing.

```sh
npm install -g truffle
mkdir solidity-experiments
cd solidity-experiments/
truffle init
```

Truffle will create all the files for an example project, including contracts
for `MetaCoin.sol`, a sample token contract. 
You should be able to compile the example contracts by running `truffle compile`.
Then, to deploy the contracts to the simulated network using the `testrpc` node
we have running, you need to run `truffle migrate`.

```sh
truffle compile
    Compiling ConvertLib.sol...
    Compiling MetaCoin.sol...
    Compiling Migrations.sol...
    Writing artifacts to ./build/contracts
truffle migrate
    Using network 'development'.
    Running migration: 1_initial_migration.js
      Deploying Migrations...
      ... 0x686ed32f73afdf4a84298642c60e2002a6d0d736a5478cc8cb22a655ac018a67
      Migrations: 0xa7edbac1156f98907a24d18df8104b5b1bd7027c
    Saving successful migration to network...
      ... 0xe3bf1e50d2262d9ffb015091e5f2974c8ebe0d6fd0df97a7dbcde8a0e51c694a
    Saving artifacts...
    Running migration: 2_deploy_contracts.js
      Deploying ConvertLib...
      ... 0x2e0e6718f01d0da6da2ada13d6e4ad662c5a20e784e04c404e9d4ef1d392bdae
      ConvertLib: 0xf4388ce4d4ce8a443228d65ecfa5149205db049f
      Linking ConvertLib to MetaCoin
      Deploying MetaCoin...
      ... 0xb03a3cde0672a2bd4dda6c01dd31641d95bd680c4e21162b3370ed6db7a5620d
      MetaCoin: 0x4fc68713f7ac86bb84ac1ef1a09881a9b8d4100f
    Saving successful migration to network...
      ... 0xb9a2245c27ff1c6506c0bc6349caf86a31bc9f700388defe04566b6d237b54b6
    Saving artifacts...
```

> macOS users: Truffle is sometimes confused by `.DS_Store` files. If you get an error mentioning one of those files, just delete it.

We just deployed the sample contracts to our testrpc node. **Wohoo!** That was easy, right? Time to create our own contract now!
