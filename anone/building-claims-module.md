# Building claims module
1. __Must layout what modules to build at the start.__ 

Why so? this is starport related. If still at beginning after starport scaffold chain, it is easy to scaffold another module into chain since go module version is unchanged.

After upgrade to go module, scaffolding another module into chain will produce errors. This error is produced by incompatible go module version between scaffolded module and current chain. Fixing is very gruesome. 

Therefore, it is best to plan out what modules intended to build in the very beginning.

2. __First thing to build: proto__

Proto will determine module messages.

Everything else inside module will based on these messages. 

A worse scenario would be building module first before building Msg proto. After that, if message is incorrect, one will have to build proto again and all module code structure that depends on that incorrect Msg.

3. __Second thing to build: function logic in keeper package__

keeper package will store all module logic. 

## Logic will often has to interact with functions from other modules. 
* Identify  which functions needed from other module M.
* Include that module M keeper into our module Keeper

```
Keeper struct {
	// we need accountKeeper to identify account of user to send money to, various account logic is needed
	accountKeeper types.AccountKeeper
	// bankKeeper is for minting coins to a module, and send money to address
	bankKeeper    types.BankKeeper
	// distrKeeper is for funding coins to community pool
	distrKeeper   types.DistrKeeper
}
```

* Those third party keepers above will be assigned in app.go

## Logic will often need to check state of module through params
* Identify which states are needed in a module
* Create a structure for states in proto/module/params.proto

```
message Params {
  option (gogoproto.goproto_stringer) = false;
  bool airdrop_enabled = 1;

  google.protobuf.Timestamp airdrop_start_time = 2 [
    (gogoproto.stdtime) = true,
    (gogoproto.nullable) = false,
    (gogoproto.moretags) = "yaml:\"airdrop_start_time\""
  ];
}
```

* Generate params.pb.go in module/types using protoc-gen-gocosmos. params.pb.go contains structure of states in go language. 
* The next task would be to register state value into a place. This place is ParamsKeeper's subspace. Include a subspace container for a module through keeper:

```
Keeper struct {
		// paramstore here is a subspace container used for containing params specific and privately to this module
		paramstore paramtypes.Subspace
	}
```

* Subspace instance will be assigned when in app.go
* State will be assigned to param subspace in module/types/params.go. Each Param pair will be a Key - Value store. In the following example, KeyEnabled will be key to state AirdropEnabled. "validateEnabled" is a function that checks if value of state AirdropEnabled is valid.

```
// ParamSetPairs - Implements params.ParamSet
// Register keys to be stored in params
func (p *Params) ParamSetPairs() paramtypes.ParamSetPairs {
	return paramtypes.ParamSetPairs{
		paramtypes.NewParamSetPair(KeyEnabled, &p.AirdropEnabled, validateEnabled),
	}
}

func validateEnabled(i interface{}) error {
	_, ok := i.(bool)
	if !ok {
		return fmt.Errorf("invalid parameter type: %T", i)
	}
	return nil
}
```

* get state value from param subspace:

```
params := k.GetParams(ctx)
```

* set state value to param subspace:

```
k.SetParams(ctx, types.DefaultParams())
```

## Logic will often need to store value
* each module will be assigned a specific storeKey. This storekey is required to access KVStore reserved only for a module. This storeKey is in module Keeper

```
Keeper struct {
	cdc      codec.BinaryCodec
	storeKey sdk.StoreKey
}
```

* specific instance of key will be assigned in app.go
* access KVStore using store key:

```
store := ctx.KVStore(k.storeKey)

//prefixStore: https://docs.cosmos.network/v0.44/core/store.html#prefix-store
prefixStore := prefix.NewStore(store, types.ClaimRecordsStorePrefix)
```

* get bytes data value from KVStore with key:

```
bz := prefixStore.Get(addr)
```

* marshall bytes data value into struct for easy processing:

```
// claimRecord is a structure
claimRecord := types.ClaimRecord{}
err := proto.Unmarshal(bz, &claimRecord)
if err != nil {
	return types.ClaimRecord{}, err
}
```

4. __Third thing to remember: Event Emitter__

For every final actions (send coins, mint coins, burn coins, claim rewards, ...), an event should be emitted so that client subscribed through WebSocket can catch these events realtime.

```
// emit a claim event with attr {sender, amount}
ctx.EventManager().EmitEvents(sdk.Events{
	sdk.NewEvent(
		"claim",
		sdk.NewAttribute(sdk.AttributeKeySender, addr.String()),
		sdk.NewAttribute(sdk.AttributeKeyAmount, claimableAmount.String()),
	),
})
```


