---
id: ethereum-basics
title: Ethereum basics
---

## Ethereum
Ethereum is an open-source public platform, distributed and based on blockchain
technology, to run applications without censorship or third-party
interference.

## The Ethereum Virtual Machine
Ethereum was designed as a smart contract platform. Its origin is actually
linked to a critique made by Vitalik Buterin on bitcoin as a very limited smart
contract platform. The [Ethereum Virtual Machine
(EVM)](http://gavwood.com/Paper.pdf) is where smart contracts run in Ethereum.
It provides a more expressive and complete language than bitcoin for scripting.
In fact, it is a [Turing Complete](https://github.com/ethereum/wiki/wiki/White-Paper#computation-and-turing-completeness)
programming language.  A good metaphor is that the EVM is a distributed global
computer where all smart contracts are executed.

## Gas
Since smart contracts run on the EVM, whose every single operation is executed
by every node in the network, there must be a mechanism to limit the
resources used by each contract. Otherwise, it could starve the whole network’s
computing power. This is why gas exists – it fixes a cost for every
instruction, like data reads and writes, expensive computations like cryptographic primitives,
making calls (sending messages) to other contracts, etc.

Each gas unit consumed by a transaction must be paid for in ether, 
based on a gas/ether price which changes dynamically. This price
is deducted from the Ethereum account sending the transaction. Transactions
also have a gas limit parameter that is an upper bound on how much gas the
transaction can consume, and is used as a safe-guard against programming errors
that could deplete an account’s funds. 

The fee paid to execute a transaction is calculated by multiplying the gas
cost and the gas price (the resulting fee is paid in ETH). You can set the gas
price for your transaction to any value, but if the gas price is too low,
no one is going to execute your code.

Every transaction is sent to the network with a gas budget. If it runs out,
the transaction will fail but will still get mined into the blockchain.

You can read more about gas
[here](https://ethereum.gitbooks.io/frontier-guide/content/costs.html).

## Accounts

Every account is identified by an address. There are two kinds of accounts
sharing the same address space: external accounts,
controlled by public-private key pairs, commonly owned by people to store ETH,
and have contract accounts, controlled by the code that they're stored with. 
The most important difference between these two is that only external accounts
can initiate transactions.

## Transactions
A transaction is a message sent from one account to another account. You can
send a transaction to another external account in order to transfer ETH. If the
target account is a contract account, its code will be executed as well. Note
that every transaction that involves code execution will be executed on all
nodes of the network. Furthermore, every code run, or transaction execution,
will be recorded in the Ethereum blockchain.

## Solidity
Solidity is a contract-oriented high-level language, with similar syntax to
JavaScript. It is statically typed, supports inheritance, libraries and complex
user-defined types. It compiles to EVM assembly, which is run by the nodes.
