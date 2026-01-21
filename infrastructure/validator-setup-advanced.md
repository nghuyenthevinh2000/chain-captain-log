# Validator setup advanced guide

## I. Setup
1. archlinux
2. [validator env](https://github.com/spf13/viper#working-with-environment-variables)
   
```bash
PREFIX_P2P_LADDR=tcp://0.0.0.0:26656
PREFIX_RPC_LADDR=tcp://0.0.0.0:26657
PREFIX_GRPC_ENABLE=false
PREFIX_GRPC_WEB_ENABLE=false
PREFIX_P2P_MAX_NUM_INBOUND_PEERS=200
PREFIX_P2P_MAX_NUM_OUTBOUND_PEERS=50
PREFIX_TX_INDEX_INDEXER=null
```
