# Fork mess

Upgrading the chain multiple times through fork (not store breaking) without clear registration can be really messy.

During the preceding of cosmwasm parity upgrade, multiple fork versions have been created to fix rebels - 2 at different heights
* https://github.com/classic-terra/core/tree/v2.1.0-rc.1
* https://github.com/classic-terra/core/releases/tag/v2.1.0-rebels2
* https://github.com/classic-terra/core/tree/v2.1.0-rebels3

Each of these comes with different wasmd version.

When trying to sync mantlemint, app hash happens. Multiple versions of terrad and wasmd are tried to no avail.

# Potential cause
When syncing nodes, blocks will be replayed which produces an app hash. The correct binary has to be used to replay blocks. Else, it will lead to different local app hash from the one recorded on - chain.

For example, this is a scenario when fork mess happens. Fork logic in binary B is applied at block 3, but local node still uses binary A. This will lead to consensus break due to mismatched app hash.

| blocks | local node | on - chain |
|--------|------------|------------|
| 1      | A          | A          |
| 2      | A          | A          |
| 3      | A          | B          |

This applies to mantlemint syncing, it has wrong binary at block 14613953. App hash consistenly happens around this height.

The goal is to find the correct combination of binary version for terrad and wasmd. One upgrade and two forks happen. The quest takes up so much time waiting for node to sync.

# Solution
The solution is to apply full upgrade on another network bajor-1, and then test mantlemint on it. For rebels-2, it will need a new genesis restart.

A fork management logic is put in place for future fork coordination: https://github.com/classic-terra/core/issues/245