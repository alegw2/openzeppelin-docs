---
id: about-openzeppelin
title: About OpenZeppelin
---

OpenZeppelin is a library for writing secure [smart contracts](https://en.wikipedia.org/wiki/Smart_contract) on Ethereum, in the [Solidity language](https://solidity.readthedocs.io/en/develop/). With OpenZeppelin, you can build distributed applications, protocols and
organizations taking advantage of good [code patterns](code-patterns.md).

> New to smart contract development? Read our [step-by-step guide](start-here.md).

The OpenZeppelin community is focused on providing short, secure smart contracts that serve as building blocks for complex
decentralized applications. Our contracts have been widely reviewed and safely manage millions of dollars' worth
of digital assets every day. 


## Quick Start

OpenZeppelin integrates with [Truffle](https://github.com/ConsenSys/truffle), an Ethereum development environment. Please install Truffle and initialize your project with `truffle init`.

```sh
npm install -g truffle
mkdir myproject && cd myproject
truffle init
```

To install the OpenZeppelin library, run the following in your Solidity project root directory:
```sh
npm init -y
npm install -E zeppelin-solidity
```

**Note that OpenZeppelin does not currently follow semantic versioning.** You may encounter breaking changes upon a minor version bump. We recommend pinning the version of OpenZeppelin you use, as done by the `-E` (`--save-exact`) option.

After that, you'll get all the library's contracts in the `node_modules/zeppelin-solidity/contracts` folder. You can use the contracts in the library like so:

```js
import 'zeppelin-solidity/contracts/ownership/Ownable.sol';

contract MyContract is Ownable {
  ...
}
```

## Security Notice
Please [follow best practices](principles.md) and use common sense when handling with real money.
OpenZeppelin is provided as-is, and we assume no responsibility for other
parties' implementation decisions. The simplest change in a contract can have
unintended consequences.

If you find a security issue in our codebase, please email [security@openzeppelin.org](mailto:security@openzeppelin.org).

## Developer Resources
To get acquainted with smart contract development, we strongly recommend
following the [step-by-step guide](start-here.md) and reading the [security reference](principles.md) carefully.

- Ask for help and keep in touch through our [Community Slack](https://slack.openzeppelin.org)
- [Framework proposal and roadmap](https://medium.com/zeppelin-blog/zeppelin-framework-proposal-and-development-roadmap-fdfa9a3a32ab)
- [Issue tracker](https://github.com/OpenZeppelin/zeppelin-solidity/issues)
- [Contribution guidelines](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/CONTRIBUTING.md)
- [Wiki](https://github.com/OpenZeppelin/zeppelin-solidity/wiki)


## Collaborating organizations and audits by OpenZeppelin

Many organizations have relied on OpenZeppelin code to build their decentralized
applications. Here are a few of them:

- [Golem](https://golem.network/)
- [Mediachain](http://www.mediachain.io/)
- [Truffle](http://truffleframework.com/)
- [Firstblood](https://firstblood.io/)
- [Rootstock](https://www.rsk.co/)
- [Consensys](https://consensys.net/)
- [DigixGlobal](https://www.dgx.io/)
- [Coinfund](https://coinfund.io/)
- [DemocracyEarth](https://democracy.earth/)
- [Signatura](https://signatura.co/)
- [Ether.camp](http://www.ether.camp/)
- [Aragon](https://aragon.one/)
- [Wings](https://wings.ai/)

## License

Code released under the [MIT License](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/LICENSE).

