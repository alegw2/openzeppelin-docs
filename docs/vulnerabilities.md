---
title: Types of vulnerabilities
id: vulnerabilities
---

## Reentrancy
Do not perform external calls in contracts. If you do, ensure that they are the very last thing you do.

## Send can fail
When sending money, your code should always be prepared for the send function to fail.

## Loops can trigger gas limit
Be careful when looping over state variables, which can grow in size and make gas consumption hit the limits.

## Timestamp dependency
Do not use timestamps in critical parts of the code, because miners can manipulate them.
