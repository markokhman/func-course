# TVM and memory layout

###### tags: `Chapter 2`

This lesson delves into the execution process and memory layout of contracts within the TON platform. The chapter provides an in-depth exploration of contract aspects, design, and functionality at the blockchain's low level. 

## Introduction to TON Contracts

Contracts in TON encapsulate data, code, and a coin balance. Each contract instance operates within its own isolated state, not able to access other contracts' states. Contracts communicate through messages, which activate the contract upon reception. Messages carry structured data that the contract processes, performs computations on, updates its state, and sends outgoing messages to other contracts.

## Understanding TVM

TVM (TON Virtual Machine) executes contract code per message, aiding local testing and auditing with its per-message instantiation.
TVM, a bytecode machine, relies on cells for TON's data layout, enabling Merkle trees and deduplication. Cell limits influence design choices for efficient data usage. Cell limits guide design, encouraging localized data and efficient proofs.


## Conclusion 

In summary, contracts in TON encompass data and code, executed by TVM. TVM is instantiated per message, ensuring secure, flexible contract execution. The TVM works with cells and integers, while other data types like continuations and bags of cells are also part of the system. The use of cells as the building blocks provides flexibility and enables data optimization.
