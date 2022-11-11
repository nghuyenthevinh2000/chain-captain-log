# Building claims module
1. __Must layout what modules to build at the start.__ 

Why so? this is starport related. If still at beginning after starport scaffold chain, it is easy to scaffold another module into chain since go module version is unchanged.

After upgrade to go module, scaffolding another module into chain will produce errors. This error is produced by incompatible go module version between scaffolded module and current chain. Fixing is very gruesome. 

Therefore, it is best to plan out what modules intended to build in the very beginning.

2. __First thing to build: proto__

Proto will determine module messages.

Everything else inside module will based on these messages. 

A worse scenario would be building module first before building Msg proto. After that, if message is incorrect, one will have to build proto again and all module code structure that depends on that incorrect Msg.

3. __Second thing to register: message codec (compressor/decompressor)__

After proto message is generated from the above step, developer needs to register that proto message to codec LegacyAmino and InterfaceRegistry. All registration is in file x/module/types/codec.go

* __InterfaceRegistry__ is used by Protobuf codec to handle interfaces that are encoded and decoded (we also say "unpacked") using google.protobuf.Any
* __LegacyAmino__ is old codec but is still used for backwards-compatibility

## InterfaceRegistry register

* Proto msg structure needs to implement "github.com/cosmos/cosmos-sdk/types.Msg" interface compliant to "github.com/cosmos/cosmos-sdk/codec/types.InterfaceRegistry.RegisterImplementations()"

```
Msg interface {
	proto.Message

	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() error

	// Signers returns the addrs of signers that must sign.
	// CONTRACT: All signatures must be present to be valid.
	// CONTRACT: Returns addrs in some deterministic order.
	GetSigners() []AccAddress
}
```

To do so, you will have to write implementation for proto Msg (for example MsgClaimFor) to comply to "github.com/cosmos/cosmos-sdk/types.Msg"

```
func (msg *MsgClaimFor) ValidateBasic() error {
	_, err := sdk.AccAddressFromBech32(msg.Sender)
	if err != nil {
		return sdkerrors.Wrapf(sdkerrors.ErrInvalidAddress, "invalid sender address (%s)", err)
	}
	return nil
}

func (msg *MsgClaimFor) GetSigners() []sdk.AccAddress {
	sender, err := sdk.AccAddressFromBech32(msg.Sender)
	if err != nil {
		panic(err)
	}
	return []sdk.AccAddress{sender}
}
```

* Register: 
	* proto Msg to InterfaceRegistry for codec 
	* proto Msg service for handling messages

```
func RegisterInterfaces(registry cdctypes.InterfaceRegistry) {
	// register proto Msg structure to InterfaceRegistry
	registry.RegisterImplementations((*sdk.Msg)(nil),
		&MsgInitialClaim{},
		&MsgClaimFor{},
	)
	
	// register Msg service
	msgservice.RegisterMsgServiceDesc(registry, &_Msg_serviceDesc)
}
```

## LegacyAmino

```
func RegisterCodec(cdc *codec.LegacyAmino) {
	cdc.RegisterConcrete(&MsgInitialClaim{}, "claim/InitialClaim", nil)
	cdc.RegisterConcrete(&MsgClaimFor{}, "claim/ClaimFor", nil)
}
```

4. __Third thing to build: function logic in keeper package__

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

5. __Fourth thing to remember: Event Emitter__

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

6. __Fifth thing to remember: client/cli__

client and cli should be easy to deal with once looked at Osmosis source code

7. __Optional: Hooks__

Here is how hook works:

Module A will hook to module B: 

```
// 1. define hook function on module A in x/moduleA/keeper
funcHook()

// 2. register module A to module B hooks in app.go:
app.StakingKeeper = *stakingKeeper.SetHooks(
	stakingtypes.NewMultiStakingHooks(app.DistrKeeper.Hooks(), app.SlashingKeeper.Hooks(), app.moduleA.Hooks()),
)

// 3. when an event funcHook on module B is fired, all registered keepers that define funcHook() will be go through:
// * DistrKeeper.funcHook()
// * SlashingKeeper.funcHook()
// * moduleA.funcHook()

```

module A's necessary logic can be executed right after event funcHook of module B.
