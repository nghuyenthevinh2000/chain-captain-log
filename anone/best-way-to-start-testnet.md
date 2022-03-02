# Best way to launch a testnet
Waiting for consensus is a gruesome task. 
Some validators will double signed, some validators will get tombstoned (a state in which a validator is killed by chain)

## Tombstone
(Tombstone caps)[https://docs.cosmos.network/master/modules/slashing/01_concepts.html#tombstone-caps]
(Staking Tombstone)[https://docs.cosmos.network/master/modules/slashing/07_tombstone.html#staking-tombstone]

A lot of shit stuffs can happen if:
1. binary is wrong
2. genesis is wrong

Leading to a whole bunch of weird problems

# Solution (only doable for testnet launch)
By starting a chain with 1 validator, consensus state is guaranted to reach right at beginning.
1. Create a bot to give faucet tokens to validators so that they can join in validating
2. Ask to set up validators using faucet distributed by bot

With this way, a testnet launch is guaranted to reach consensus and success.