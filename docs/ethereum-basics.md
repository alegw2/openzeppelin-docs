---
id: ethereum-basics
title: Ethereum basics
---

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
Since smart contracts run on the EVM, there must be a mechanism to limit the
resources used by each contract. Every single operation that is executed inside
the EVM is actually simultaneously executed by every node in the network. This
is why gas exists. An Ethereum transaction contract code can trigger data reads
and writes, do expensive computations like using cryptographic primitives, make
calls (send messages) to other contracts, etc. Each of these operations have a
cost measured in gas, and each gas unit consumed by a transaction must be paid
for in Ether, based on a gas/Ether price which changes dynamically. This price
is deducted from the Ethereum account sending the transaction. Transactions
also have a gas limit parameter that is an upper bound on how much gas the
transaction can consume, and is used as a safe-guard against programming errors
that could deplete an account’s funds. You can read more about gas
[here](https://ethereum.gitbooks.io/frontier-guide/content/costs.html).
