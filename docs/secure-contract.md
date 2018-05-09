---
id: secure-contract
title: Writing a secure contract
---

Emin Gün Sirer said it well: [smart contracts are pretty difficult to get right](http://hackingdistributed.com/2016/07/13/reentrancy-woes/).
This section will take you through problems that typically arise when beginner developers start
writing smart contract code.

## Fail as early and loudly as possible
A simple yet powerful programming 
good practice is to [make your code fail as promptly as possible](https://oncodingstyle.blogspot.com.ar/2008/10/fail-early-fail-loudly.html). And be loud about it. Let’s see an example of a function that behaves timidly:

```
// UNSAFE CODE, DO NOT USE!
contract BadFailEarly {
  uint constant DEFAULT_SALARY = 50000;
  mapping(string => uint) nameToSalary;
  function getSalary(string name) constant returns (uint) {
    if (bytes(name).length != 0 && nameToSalary[name] != 0) {
      return nameToSalary[name];
    } else {
      return DEFAULT_SALARY;
    }
  }
}
```

We want to avoid a contract failing silently, or continuing execution in an
unstable or inconsistent state. The function getSalary is checking for
conditions before returning the stored salary, which is a good thing. The
problem is, in case those conditions are not met, a default value is returned.
This could hide an error from the caller. This is an extreme case, but this
kind of programming is very common, and normally arises from fear of errors
breaking our app. Truth is, the sooner we fail, the easier it will be to find
the problem. If we hide errors, they can propagate to other parts of the code
and cause inconsistencies which are difficult to trace. Here's a more correct
approach, that also takes advantage of another programming best practice:

## Separate conditionals to easily identify which fails
```
contract GoodFailEarly {
  mapping(string => uint) nameToSalary;
  
  function getSalary(string name) constant returns (uint) {
    if (bytes(name).length == 0) throw;    
    if (nameToSalary[name] == 0) throw;
    
    return nameToSalary[name];
  }
}
```

Note that some of these checks (especially those depending on internal state) 
can be implemented via Function Modifiers.

## Favor pull over push payments
Every ether transfer implies potential code execution. The receiving address
can implement a [fallback function](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function) 
that can throw an error. Thus, we should
never trust that a send call will execute without error. A solution: our
contracts should [favor pull over push](https://github.com/ethereum/wiki/wiki/Safety#favor-pull-over-push-for-external-calls) 
for payments. Take a look at this innocent-looking code for a bidding function:

```
// UNSAFE CODE, DO NOT USE!
contract BadPushPayments {
  address highestBidder;
  uint highestBid;
 
  function bid() {
    if (msg.value < highestBid) throw;
    if (highestBidder != 0) {
      // return bid to previous winner
      if (!highestBidder.send(highestBid)) {
        throw;
      }
    }
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
}
```

Note that the contract calls the `send` function and checks its return value,
which seems reasonable. But it calls `send` in the middle of a function, which is
unsafe. Why? Remember that, as stated above, `send` can trigger the execution of
code in another contract.

Imagine someone bids from an address which simply throws an error every time
someone sends money to it. What happens when someone else tries to outbid that?
The `send` call will always fail, bubbling up and making bid throw an exception.
A function call that ends in error leaves the state unchanged (any changes made
are rolled back). That means nobody else can bid, and the contract is broken.

The easiest solution is to separate payments into a different function, and
have users request (pull) funds independently of the rest of the contract
logic:

```
contract GoodPullPayments {
  address highestBidder;
  uint highestBid;
  mapping(address => uint) refunds;
  
  function bid() external {
    if (msg.value < highestBid) throw;
    
    if (highestBidder != 0) {
      refunds[highestBidder] += highestBid;
    }
    
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
  
  function withdrawBid() external {
    uint refund = refunds[msg.sender];
    refunds[msg.sender] = 0;
    if (!msg.sender.send(refund)) {
      refunds[msg.sender] = refund;
    }
  }
}
```

This time, we use a mapping to store refund values for each outbid bidder, and
provide a function to withdraw their funds. In case of a problem in the send
call, only that bidder is affected. This is a simple pattern that solves many
other problems (such as [reentrancy](vulnerabilities.md)), so remember: when sending ether,
favor pull over push payments.

OpenZeppelin has 
[a contract you can inherit from to easily use this pattern](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/payment/PullPayment.sol).

## Order your function code: conditions, then actions, then interactions
As an extension of the fail-early principle, a good practice is to structure
all your functions as follows: first, check all the pre-conditions; then, make
changes to your contract’s state; and finally, interact with other contracts.

Conditions, actions, interactions. Sticking to this function structure will
save you lots of problems. Let’s see an example of a function using this
pattern, [taken from the Solidity docs](https://solidity.readthedocs.io/en/latest/solidity-by-example.html).

```
function auctionEnd() {
  // 1. Conditions
  if (now <= auctionStart + biddingTime)
    throw; // auction did not yet end
  if (ended)
    throw; // this function has already been called

  // 2. Actions
  ended = true;
  AuctionEnded(highestBidder, highestBid);

  // 3. Interactions
  if (!beneficiary.send(highestBid))
    throw;
  }
}
```

## Be aware of the platform limits
The EVM has a lot of hard limits on what our contracts can do. These are
platform-level security considerations, but may threaten your particular
contract’s security if you don’t know about them. Let’s take a look at the
following innocent-looking employee bonus management code:

```
// UNSAFE CODE, DO NOT USE!
contract BadArrayUse {
  
  address[] employees;
  
  function payBonus() {
    for (var i = 0; i < employees.length; i++) {
      address employee = employees[i];
      uint bonus = calculateBonus(employee);
      employee.send(bonus);
    }     
  }
  
  function calculateBonus(address employee) returns (uint) {
    // some expensive computation ...
  }
}
```

Read the code: it’s pretty straightforward and seems correct. If you knew the
platform limits well, though, you'd discover 3 potential problems.

The first is that the type of `i` will be `uint8`, because this is the
smallest type that is required to hold the value 0. If the array has more than
255 elements, the loop will not terminate, resulting in gas depletion. Better
use the explicit type uint for no surprises and higher limits. Avoid declaring
variables using var if possible. Let’s fix that:

```
// STILL UNSAFE CODE, DO NOT USE!
contract BadArrayUse {
  
  address[] employees;
  
  function payBonus() {
    for (uint i = 0; i < employees.length; i++) {
      address employee = employees[i];
      uint bonus = calculateBonus(employee);
      employee.send(bonus);
    }     
  }
  
  function calculateBonus(address employee) returns (uint) {
    // some expensive computation ...
  }
}
```

The second thing you should consider is the gas limit. Gas is Ethereum’s
mechanism to charge for network resources. Every function call that modifies
state has a gas cost. Imagine calculateBonus calculates the bonus for each
employee based on some complex computation like calculating the profit over
many projects. This would spend a lot of gas, which could easily reach the
transaction’s or block’s gas limit. If a transaction reaches the gas limit, all
changes will be reverted but the fee is still paid. Be aware of variable gas
costs when using loops. Let’s optimize the contract by separating the bonus
calculation from the for loop. Please note that this still has the issue that
as the employees array grows, the gas cost grows.

```
// UNSAFE CODE, DO NOT USE!
contract BadArrayUse {
  
  address[] employees;
  mapping(address => uint) bonuses;  
  
  function payBonus() {
    for (uint i = 0; i < employees.length; i++) {
      address employee = employees[i];
      uint bonus = bonuses[employee];
      employee.send(bonus);
    }     
  }
  
  function calculateBonus(address employee) returns (uint) {
    uint bonus = 0;
    // some expensive computation modifying the bonus...
    bonuses[employee] = bonus;
  }
}
```

The EVM’s call stack has a hard limit of 1024. [EIP150](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-150.md) fixed
a related vulnerability by reducing the amount of gas each
recursive call gets, but you should still be aware of the limit for your code
to work as intended.

Here’s the final version of the code that fixes all these issues:
```
import './PullPayment.sol';
contract GoodArrayUse is PullPayment {
  address[] employees;
  mapping(address => uint) bonuses;
  
  function payBonus() {
    for (uint i = 0; i < employees.length; i++) {
      address employee = employees[i];
      uint bonus = bonuses[employee];
      asyncSend(employee, bonus);
    }
  }
  function calculateBonus(address employee) returns (uint) {
    uint bonus = 0;
    // some expensive computation...
    bonuses[employee] = bonus;
  }
}
```

To sum up, be sure to remember about (1) limits in the types you’re using, (2)
limits in the gas costs of your contract, and (3) the call stack depth limit.

## Write tests
Writing tests is a lot of work, but will save you 
from [regression problems](https://en.wikipedia.org/wiki/Software_regression). A
regression bug appears when a previously correct component gets broken based on
a recent change.

For more information about testing, check out [Truffle’s guide](https://blog.zeppelin.solutions/onward-with-ethereum-smart-contract-security-97a827e47702).

## Implement fault tolerance and automatic bug bounties
Read [this excellent piece by Peter Borah](https://medium.com/@peterborah/we-need-fault-tolerant-smart-contracts-ec1b56596dbc).

Code reviews and security audits are not enough to be safe. Our code needs to be ready for the worst. In case there is a vulnerability in our smart contract, there should be a way for it to safely recover. Not only that, but we should try to find those vulnerabilities as early as possible. That’s where automatic bug bounties built into our contract can help.

Let’s take a look at this simple implementation of an automatic bug bounty for a hypothetical Token contract:
```
import './PullPayment.sol';
import './Token.sol';
contract Bounty is PullPayment {
  bool public claimed;
  mapping(address => address) public researchers;
  
  function() {
    if (claimed) throw;
  }
  
  function createTarget() returns(Token) {
    Token target = new Token(0);
    researchers[target] = msg.sender;
    return target;
  }
  
  function claim(Token target) {
    address researcher = researchers[target];
    if (researcher == 0) throw;
    
    // check Token contract invariants
    if (target.totalSupply() == target.balance) {
      throw;
    }
    asyncSend(researcher, this.balance);
    claimed = true;
  }
}
```

As before, we’re using PullPayment to make our outgoing payments safe. This
Bounty contract allows researchers to create copies of the Token contract we
want audited. Anyone can contribute to the bug bounty by sending transactions
to the Bounty contract’s address. If any researcher manages to corrupt his copy
of the Token contract, making some invariant break (for example, in this case,
making the total supply of tokens different from the Token’s balance), he’ll
get the bounty reward. Once the bounty is claimed, the contract won’t accept
any more funds (that nameless function is called the contract’s fallback
function, and is executed every time the contract is sent money directly).

As you can see, this has the nice property that it is a separate contract and
requires no modification of our original Token contract. OpenZeppelin has a
[full implementation here](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/Bounty.sol).

As for fault tolerance, we will need to modify our original contract to add
extra safety mechanisms. A simple idea is to allow a contract’s curator to
freeze the contract as an emergency mechanism. Let’s implement
this behavior via inheritance:
```
contract Stoppable {
  address public curator;
  bool public stopped;
  modifier stopInEmergency { if (!stopped) _; }
  modifier onlyInEmergency { if (stopped) _; }
  
  function Stoppable(address _curator) {
    if (_curator == 0) throw;
    curator = _curator;
  }
  
  function emergencyStop() external {
    if (msg.sender != curator) throw;
    stopped = true;
  }
}
```

Stoppable allows one to specify a curator address that can stop the contract.
What does “stopping the contract” mean? That’s to be defined by the child
contract inheriting from Stoppable by using the function modifiers
`stopInEmergency` and `onlyInEmergency`. Let’s see an example:

```
import './PullPayment.sol';
import './Stoppable.sol';
contract StoppableBid is Stoppable, PullPayment {
  address public highestBidder;
  uint public highestBid;
  
  function StoppableBid(address _curator)
    Stoppable(_curator)
    PullPayment() {}
  
  function bid() external stopInEmergency {
    if (msg.value <= highestBid) throw;
    
    if (highestBidder != 0) {
      asyncSend(highestBidder, highestBid);
    }
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
  
  function withdraw() onlyInEmergency {
    suicide(curator);
  }
}
```

In this toy example, the bid can now be stopped by a curator, defined when the
contract is created. While the StoppableBid is in normal mode, only the bid
function can be called. If something weird happens and the contract is in an
inconsistent state, the curator can step in and activate the emergency state.
This makes the bid function uncallable, and allows the function withdraw to
work.

In this case, emergency mode would only allow the curator to destroy the
contract and recover the funds, but in a real case, recovery logic could be
more complex (for example, returning funds to their owners). Here's the
OpenZeppelin [implementation of Stoppable](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/lifecycle/Pausable.sol),
that we renamed Pausable.

## Limit the amount of funds deposited
Another way to protect our smart contracts from attacks is to limit their
scope. Attackers will most probably target high-profile contracts managing
millions of dollars. Not all smart contracts need to have such high stakes.
Especially if we’re conducting experiments. In such cases, it might be useful
to limit the amount of funds our contract accepts. This is as simple as a hard
limit on the balance of the contract’s address.

Here’s a simplified example on how to do this:
```
contract LimitFunds {
  
  uint LIMIT = 5000;
  
  function() { throw; }
  
  function deposit() {
    if (this.balance > LIMIT) throw;
    ...
  }
}
```

The short fallback function will reject any direct payments to the contract.
The deposit function will first check if the contract’s balance exceeds the
desired limit, or throw an exception. More interesting things like dynamic or
managed limits are easy to implement too.

## Write simple and modular code

Security is a match between our intention and what our code actually does. This
is very hard to verify, especially if the codebase is huge and messy. That’s why
it’s important to write simple and modular code.

This means, functions should be as short as possible, code dependencies should
be reduced to the minimum, and files should be as small as possible, separating
independent logic into modules, each with a single responsibility.

Taking care when naming is also one of the best ways to express our intention when coding. Think
a lot about the names you chose, to make your code as clear as possible.

Let’s study an example of bad naming of
[Events](https://solidity.readthedocs.io/en/latest/contracts.html#events).
[This function from the DAO](https://github.com/slockit/DAO/blob/develop/DAO.sol#L618-L691) is too long
and complex. Try to keep your functions much shorter, say, up to 30 or 40 lines of code max.
Ideally, you should be able to read functions and understand what they do in
less than a minute. Another problem is the bad naming for the event Transfer in
line [685](https://github.com/slockit/DAO/blob/develop/DAO.sol#L685). 
The name differs from [a function called transfer](https://github.com/slockit/DAO/blob/develop/DAO.sol#L755)
by only 1 character!  This is inviting confusion for everyone. In general, the recommended naming for
events is that they should start with “Log”. In this case, a better name would be LogTransfer.


Remember, write your contracts as simple, modular, and well-named as possible.
This will greatly facilitate others and yourself in auditing your code.

## Don't write all your code from scratch
Finally, as the old adage reads: [“Don’t roll your own
crypto”](https://security.stackexchange.com/questions/18197/why-shouldnt-we-roll-our-own). 
This especially applies to smart contract code. You’re dealing with money, your code and data
is public, and you’re running in a new and experimental platform. The stakes
are high and the chances to mess-up are everywhere.

These practices help secure our smart contracts. But ultimately, we should
create better developer tools to build smart contracts. There are some
interesting initiatives including [better type systems](https://www.youtube.com/watch?v=H2uwUdzVD9I), 
[Serenity Abstractions](https://blog.ethereum.org/2015/12/24/understanding-serenity-part-i-abstraction/),
and the [Rootstock platform](https://www.rsk.co/).

There’s lots of good and secure code already written and frameworks are
starting to appear. We’ve baked the industry best practices into our
[OpenZeppelin Solidity repo](https://github.com/OpenZeppelin/openzeppelin-solidity), 
and we recommend you use it as the building blocks for your more complex applications.

## Remember!
Before ever writing code that handles real money, read more about the [types of vulnerabilities](vulnerabilities.md),
and ask for help at the [OpenZeppelin community channel](https://slack.openzeppelin.org/).
