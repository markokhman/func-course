# Review of classic business problems

###### tags: `Chapter 2`

- **ERC20 vs Jetten Standard:**
  - ERC20 token standard in Ethereum is limited in scalability due to individual contracts for each token.
  - Jetton standard in TON proposes a scalable approach using separate contracts for each user's balance, avoiding central contract congestion.

- **Multisig Contracts:**
  - Traditional multisig contracts may face challenges with gas fees and scalability.
  - Scalable multisig solution involves creating separate request contracts, allowing users to add their votes, and resolving based on threshold.

- **Plugins in Wallets:**
  - Wallets can delegate functionalities to plugins, easing subscription payments and customizations.
  - Scalability can be achieved by listing supported plugin codes rather than storing individual contract addresses.

- **Staking Pool:**
  - Staking pool with validation cycles manages deposits, withdrawals, and payouts.
  - Scalable solution involves using separate contracts for deposits, withdrawals, and payouts per user per cycle.

- **Conclusion:**
  - Scalability is achieved by using individual contracts for each data element instead of lists or databases.
  - Concepts like parent-child authentication or sibling authentication are utilized for efficient communication and verification.


