---
id: first-contract
title: Writing your first contract
---

We’ll be writing a simple Proof of Existence smart contract. The idea is to
create a digital notary that stores hashes of documents as proofs of their
existence. Using Truffle:

```sh
truffle create contract ProofOfExistence1
```

Now open `contracts/ProofOfExistence1.sol` in your favorite text editor (I use
vim with Solidity syntax highlighting), and paste this initial version of the
code:

```
pragma solidity ^0.4.15;
// Proof of Existence contract, version 1
contract ProofOfExistence1 {
  // state
  bytes32 public proof;
  // calculate and store the proof for a document
  // *transactional function*
  function notarize(string document) {
    proof = proofFor(document);
  }
  // helper function to get a document's sha256
  // *read-only function*
  function proofFor(string document) constant returns (bytes32) {
    return sha256(document);
  }
}
```

We’ll start with something simple but incorrect and move towards a better
solution. This is a definition of a Solidity contract, which is like a class in
other programming languages. Contracts have state and functions. It is
important to distinguish two kinds of functions that can appear in a contract:

- **Read-only (constant) functions** don’t perform any state
  changes. They only read state, perform computations, and return values. As
these functions can be resolved locally by each node, they cost no gas. Marked
with the keyword `constant`.
- **Transactional functions** perform a state change in the contract or move
  funds. As these changes need to be reflected in the blockchain, transactional
function execution requires sending a transaction to the network and spending
gas.

Our contract above has one of each kind, marked in the documentation. We’ll see
how the kind of function we’re using with modifies how we interact with the
smart contract in the next section.

This simple version only stores one proof at a time, using the data type
bytes32, or 32 bytes, which is the size of a sha256 hash. The transactional
function notarize allows one to store the hash of a document in our smart
contract’s state variable `proof`. This variable is public, and it's the only way a
user of our contract has to verify if a document has been notarized. We’ll do
that ourselves shortly, but first...

Let’s deploy ProofOfExistence1 to the network! This time, you’ll have to edit
the migration file (`migrations/2_deploy_contracts.js`) to make Truffle deploy
our new contract. Replace the contents with the following:

```
var ProofOfExistence1 = artifacts.require("./ProofOfExistence1.sol");
module.exports = function(deployer) {
  deployer.deploy(ProofOfExistence1);
};
```

To make sure this migration runs again, you’ll need to use the `reset` flag.
Read more on Truffle migrations [here](http://truffleframework.com/docs/getting_started/migrations).

```sh
truffle migrate --reset
```

## Interacting with the contract

Now that our contract is deployed, let’s play with it! We can send messages to it via function calls and read its public state. We’ll use the Truffle console for that:

```sh
$ truffle console
// get the deployed version of our contract
truffle(default)> var poe = ProofOfExistence1.at(ProofOfExistence1.address)
// and print its address 
truffle(default)> poe.address
0x3d3bce79cccc331e9e095e8985def13651a86004
// let's register our first "document"
truffle(default)> poe.notarize('An amazing idea')
{ tx: '0x18ac...cb1a',
  receipt: 
   { transactionHash: '0x18ac...cb1a',
     ...
   },
  logs: [] }
// let's now get the proof for that document
truffle(default)> poe.proofFor('An amazing idea')
0xa3287ff8d1abde95498962c4e1dd2f50a9f75bd8810bd591a64a387b93580ee7
// To check if the contract's state was correctly changed:
truffle(default)> poe.proof()
0xa3287ff8d1abde95498962c4e1dd2f50a9f75bd8810bd591a64a387b93580ee7
// The hash matches the one we previously calculated
```

First thing we do is obtain a representation of our deployed contract and store
it in a variable called `poe`.

We then call the transactional function notarize, which involves a state
change. When we call a transactional function, we get a Promise that resolves
to a transaction object, not what the actual function returns. Remember that to
change the EVM state we need to spend gas and send a transaction to the
network. That’s why we get a transaction information object as the result of
the Promise, referring to the transaction that did this state change. In this
case, we are not interested in the transaction id, so we just discard the
Promise. When writing a real app, we’ll want to save it to check the resulting
transaction and catch errors.

Next, we call the read-only (constant) function proofFor. Remember to mark your
read-only functions with the keyword constant, or else Truffle will try to
craft a transaction to execute them. This is a way to tell Truffle that we’re
not interacting with the blockchain but just reading from it. By using this
read-only function, we obtain the sha256 hash of the ‘An amazing idea’
“document”.

We now need to contrast this with the state of our smart contract. To check if
the state changed correctly, we need to read the proof public state variable.
To get the value of a public state variable, we can call a function of the same
name, which returns a Promise of its value. In our case, the output hash is the
same, so everything worked as expected :)

For more info on how to interact with contracts, read this section of the
Truffle documentation.

As you can see from the snippet above, our first version of the Proof of
Existence smart contract seems to be working! Good work! It’s only good for
registering one document at a time, though. Let’s create a better version.

## Improving the code

Let’s change the contract to support multiple document proofs. Copy the
original file with the name contracts/ProofOfExistence2.sol and apply these
changes. The main changes are: we change the proof variable into a bytes32
array and call it proofs, we make it private, and we add a function to check if
a document has already been notarized by iterating that array.

```
pragma solidity ^0.4.15;
// Proof of Existence contract, version 2
contract ProofOfExistence2 {
  // state
  bytes32[] private proofs;
  // store a proof of existence in the contract state
  // *transactional function*
  function storeProof(bytes32 proof) {
    proofs.push(proof);
  }
// calculate and store the proof for a document
  // *transactional function*
  function notarize(string document) {
    bytes32 proof = proofFor(document);
    storeProof(proof);
  }
// helper function to get a document's sha256
  // *read-only function*
  function proofFor(string document) constant returns (bytes32) {
    return sha256(document);
  }
// check if a document has been notarized
  // *read-only function*
  function checkDocument(string document) constant returns (bool) {
    bytes32 proof = proofFor(document);
    return hasProof(proof);
  }
  // returns true if proof is stored
  // *read-only function*
  function hasProof(bytes32 proof) constant returns (bool) {
    for (uint256 i = 0; i < proofs.length; i++) {
      if (proofs[i] == proof) {
        return true;
      }
    }
    return false;
  }
}
```

Let’s interact with the new functions: (remember to update
`migrations/2_deploy_contracts.js` to include the new contract and 
run `truffle migrate --reset`).

```
// deploy contracts
truffle(default)>  migrate --reset
// Get the new version of the contract
truffle(default)> var poe = ProofOfExistence2.at(ProofOfExistence2.address)
// let's check for some new document, and it shouldn't be there.
truffle(default)> poe.checkDocument('hello')
false
// let's now add that document to the proof store
truffle(default)> poe.notarize('hello')
{ tx: '0x1d2d...413f',
  receipt: { ... },
  logs: []
}
// let's now check again if the document has been notarized!
truffle(default)> poe.checkDocument('hello')
true
// success!
// we can also store other documents and they are recorded too
truffle(default)> poe.notarize('some other document');
truffle(default)> poe.checkDocument('some other document')
true
```

This version is better than the first, but still has some problems. Note that
every time we want to check if a document was notarized, we need to iterate
through all existing proofs. This makes the contract spend more and more gas on
each check as more documents are added. A better structure to store proofs is a
map. Luckily, Solidity supports maps, and calls them mappings. Another thing
we’ll improve in this version is removing all that extra comments marking
read-only or transactional functions. I think you get it by now :)

Here’s the final version, which should be pretty easy to understand because you
followed along previous versions:

```
pragma solidity ^0.4.15;
// Proof of Existence contract, version 3
contract ProofOfExistence3 {
  mapping (bytes32 => bool) private proofs;
  // store a proof of existence in the contract state
  function storeProof(bytes32 proof) {
    proofs[proof] = true;
  }
  // calculate and store the proof for a document
  function notarize(string document) {
    var proof = proofFor(document);
    storeProof(proof);
  }
  // helper function to get a document's sha256
  function proofFor(string document) constant returns (bytes32) {
    return sha256(document);
  }
  // check if a document has been notarized
  function checkDocument(string document) constant returns (bool) {
    var proof = proofFor(document);
    return hasProof(proof);
  }
  // returns true if proof is stored
  function hasProof(bytes32 proof) constant returns(bool) {
    return proofs[proof];
  }
}
```

That looks good enough. And it works exactly as the second version. To try it out,
remember to update the migration file and run `truffle migrate --reset` again.
All the code in this tutorial can be found in 
[this GitHub repo](https://github.com/maraoz/solidity-experiments).

## Deploying to the testnet
Once you’ve tested your contract extensively using testrpc in the simulated
network, you’re ready to try it in the real network! To do that, we need a real
testnet/livenet Ethereum client. Follow these instructions to install geth.

During development, you should run nodes in testnet mode, so that you can test
everything out without risking real money. Testnet mode (also known as Ropsten) 
is identical to the real Ethereum (mainnet), but the Ether token
there has no monetary value. Don’t be lazy and remember to always develop in
testnet mode, **you will regret it if you lose real Ether due to a programming error** 
(trust me on that, hehe).

Run geth in testnet mode, with RPC server enabled:
```
geth --testnet --rpc console 2>> geth.log
```

This will open a console where you can type in basic commands to control your
node/client. Your node will begin downloading the testnet blockchain, and you
can check progress by checking eth.blockNumber. While the blockchain is
downloading, you can still run commands. For example, let’s create an account:
(remember the password!)

```
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0xa88614166227d83c93f4c50be37150b9500d51fc"
```

Let’s send some coins there and check the balance. You can get free testnet
Ether [here](https://faucet.metamask.io). Just copy-paste the address you just 
generated and this faucet will send you some testnet Ether. To check your balance, run:

```
> eth.getBalance(eth.accounts[0])
0
```

It will show no balance because your node hasn’t synced with the rest of the
network yet. While you wait for that, check your balance in a testnet block
explorer. There, you can also see the current testnet highest block number
(#1819865 at the time of writing), which you can use in combination with
eth.blockNumber to know when your node has synced completely.

Once your node has synced, you’re ready to deploy the contracts to the testnet
using Truffle. First, unlock your main geth account, so that Truffle can use
it. And be sure that it holds some balance, or you won’t be able to push a new
contract to the network. On geth run:

```
> personal.unlockAccount(eth.accounts[0], "mypassword", 24*3600)
true
> eth.getBalance(eth.accounts[0])
1000000000000000000
```

Ready to go! If some of these two are not working for you, check the steps
above and make sure you’ve completed them correctly. Now run `truffle migrate
--reset`. This time it will take longer to complete, as we’re connecting to the
actual network and not one simulated by testrpc. Once it completes, you can
interact with the contract using the same approach as before.

The testnet-deployed version of ProofOfExistence3 can be found at the address
0xcaf216d1975f75ab3fed520e1e3325dac3e79e05. (Feel free to interact with it and
send your proofs!)

I’ll leave the details on how to deploy to the live network to the reader. You
should only do this once you’ve extensively tested your contracts in simulated
and test networks. Remember [any programming error on the mainnet can result in monetary loss](https://blog.ethereum.org/2016/06/17/critical-update-re-dao-vulnerability/)!
