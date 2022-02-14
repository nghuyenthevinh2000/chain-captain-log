# Dig Mainnet Captain Log

1. source code repo: https://github.com/notional-labs/dig
2. website: https://digchain.org

## I. Captain Log 1: Experience on the first day of post dig main-net.
1. Get Wallet and Explorer up and running in test-net, not main-net.
2. Super careful with setting coin distribution (related to genesis.json)
3. Even though original goals in Dig include the two goal above, failure still happens. Why so? according to my observation, the goal is not closely monitored as environments are rapidly updated leading to a drift.

## Captain Log 2: Experience on the third day of post dig main-net
1. a single REST endpoint and RPC endpoint have to be created since the beginning till end of chain (a lot of stuff relies on an uniform, stable and long - lasting endpoint)
2. prepare also https REST and RPC endpoints for later usage

## Captain Log 3: Experience after 1 week of dig main-net
**Airdrop**: How to describe this? Airdrop is related strongly to tokenomics. Poor airdrop then the project is likely to fail.

1. Airdrop is related to genesis file in which we have to list fully airdropped accounts.
* tool is required
* it is likely to miss accounts.
* need plan for crisis in case of missed accounts.
2. Target group for airdrop will lead to different mechanism of airdropping. (Need to think of this carefully)
* In case of Dig, we have to airdrop to all DFY holders. But, DFY holders are eth-format which is incompatible to dig1. A mechanism to import eth-format leads to easier air drop but we have to build the whole infrastructure for eth-format signing.