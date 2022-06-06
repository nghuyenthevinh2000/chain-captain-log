# Sei protocol non determinism error introduces mismatched app hash error

# Error details

App hash is mismatched. This means that non - determinism error is in play. If a block is deterministic, then app hash would have been deterministic and correctly as prediction at future height.

```
3:06AM ERR prevote step: ProposalBlock is invalid err="wrong Block.Header.AppHash.  Expected 03ADB298A15906EA6C85B3FFF368332178226CF4E5AA8D748E2EB274C5A4C3CA, got 00DD4E8679DAB5C4A854C574FA06FE3EF71684E45F3CF47AF055B729A8D390C3" height=293540 module=consensus round=66
```

# Reason

Somewhere in source code has introduced non - determinism value. Storing this non - deterministic value to kv store will finalize the non - deterministic value. This will lead to app hash mismatched.

Relevant issue: https://github.com/sei-protocol/sei-chain/pull/33

In the case of Sei protocol chain, they introduce non - deterministic value through storing map value.

```
maps are hash tables, the key/value pairs are in no particular order. In fact, Go will randomize the iteration order just to make sure you don't depend on some particular ordering that may change in the future (this is another example where it's obvious this language was designed by seasoned developers who have been bitten by bugs resulting from spurious assumptions about past behavior that was not actually guaranteed). Each time you run the for...range loop above, it will list the words in a different order.
```

The above mechanism of map in golang has introduced non - deterministic value when Sei decides to use it to store value and save it to the chain.

# Solution

The way to fix is to replace map to slice data structure.

In case, map is still needed, then we will use cosmos - based KV store.
