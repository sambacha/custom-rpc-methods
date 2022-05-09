---
title: Custom EVM RPC Methods Index
extra:
  date: 2022-05-09
  version: "0.0.2"

---

## Summary <!-- omit in toc -->


Special non-standard methods that aren’t included within the original RPC specification:

### `evm_snapshot`

-   `evm_snapshot` : Snapshot the state of the blockchain at the current block. Takes no parameters. Returns the integer id of the snapshot created. A snapshot can only be used once. After a successful `evm_revert`, the same snapshot id cannot be used again. Consider creating a new snapshot after each `evm_revert` _if you need to revert to the same point multiple times_.
    
```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"id":1337,"jsonrpc":"2.0","method":"evm_snapshot","params":[]}' \
http://localhost:8545
```
    
```json
{ "id": 1337, "jsonrpc": "2.0", "result": "0x1" }
```

### `evm_revert`

-   `evm_revert` : Revert the state of the blockchain to a previous snapshot. Takes a single parameter, which is the snapshot ID to revert to. This deletes the given snapshot, as well as any snapshots taken after (Ex: reverting to id `0x1` will delete snapshots with IDs `0x1`, `0x2`, `etc`... If no snapshot ID is passed it will revert to the latest snapshot. Returns `true`.
    
```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"id":1337,"jsonrpc":"2.0","method":"evm_revert","params":["0x1"]}' \
http://localhost:8545
```
    
```json
{ "id": 1337, "jsonrpc": "2.0", "result": true }
```

### `evm_increaseTime`

-   `evm_increaseTime` : Jump forward in time. Takes one parameter, which is the amount of time to increase in seconds. Returns the total time adjustment, in seconds.
    
```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"id":1337,"jsonrpc":"2.0","method":"evm_increaseTime","params":[60]}' \
http://localhost:8545
```
    
```json
{ "id": 1337, "jsonrpc": "2.0", "result": "060" }
```

### `evm_mine`

-   `evm_mine` : Force a block to be mined. Takes one optional parameter, which is the timestamp a block should set up as the mining time. Mines a block independent of whether mining is started or stopped.
    
```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"id":1337,"jsonrpc":"2.0","method":"evm_mine","params":[1231006505000]}' \       
http://localhost:8545
```
    
```json
{ "id": 1337, "jsonrpc": "2.0", "result": "0x0" }
```

### eth_sendBundle

`ethSendBundle` can be used to send your bundles to the relay. The `eth_sendBundle` RPC has the following payload format:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendBundle",
  "params": [
    {
      txs,               // Array[String], A list of signed transactions to execute in an atomic bundle
      blockNumber,       // String, a hex encoded block number for which this bundle is valid on
      minTimestamp,      // (Optional) Number, the minimum timestamp for which this bundle is valid, in seconds since the unix epoch
      maxTimestamp,      // (Optional) Number, the maximum timestamp for which this bundle is valid, in seconds since the unix epoch
      revertingTxHashes, // (Optional) Array[String], A list of tx hashes that are allowed to revert
    }
  ]
}
```

example:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendBundle",
  "params": [
    { "txs": ["0x123abc...", "0x456def..."], "blockNumber": "0xb63dcd", "minTimestamp": 0, "maxTimestamp": 1615920932 }
  ]
}
```

example response:

```jsonc
{
  "jsonrpc": "2.0",
  "id": "123",
  "result": {
    "bundleHash": "0x2228f5d8954ce31dc1601a8ba264dbd401bf1428388ce88238932815c5d6f23f"
  }
}
```

### eth_callBundle

`eth_callBundle` can be used to simulate a bundle against a specific block number, including simulating a bundle at the
top of the next block. The `eth_callBundle` RPC has the following payload format:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_callBundle",
  "params": [
    {
      txs,               // Array[String], A list of signed transactions to execute in an atomic bundle
      blockNumber,       // String, a hex encoded block number for which this bundle is valid on
      stateBlockNumber,  // String, either a hex encoded number or a block tag for which state to base this simulation on. Can use "latest"
      timestamp,         // (Optional) Number, the timestamp to use for this bundle simulation, in seconds since the unix epoch
    }
  ]
}
```

> example:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_callBundle",
  "params": [
    {
      "txs": ["0x123abc...", "0x456def..."],
      "blockNumber": "0xb63dcd",
      "stateBlockNumber": "latest",
      "timestamp": 1615920932
    }
  ]
}
```

> example response:

```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "result": {
    "bundleGasPrice": "476190476193",
    "bundleHash": "0x73b1e258c7a42fd0230b2fd05529c5d4b6fcb66c227783f8bece8aeacdd1db2e",
    "coinbaseDiff": "20000000000126000",
    "ethSentToCoinbase": "20000000000000000",
    "gasFees": "126000",
    "results": [
      {
        "coinbaseDiff": "10000000000063000",
        "ethSentToCoinbase": "10000000000000000",
        "fromAddress": "0x02A727155aeF8609c9f7F2179b2a1f560B39F5A0",
        "gasFees": "63000",
        "gasPrice": "476190476193",
        "gasUsed": 21000,
        "toAddress": "0x73625f59CAdc5009Cb458B751b3E7b6b48C06f2C",
        "txHash": "0x669b4704a7d993a946cdd6e2f95233f308ce0c4649d2e04944e8299efcaa098a",
        "value": "0x"
      },
      {
        "coinbaseDiff": "10000000000063000",
        "ethSentToCoinbase": "10000000000000000",
        "fromAddress": "0x02A727155aeF8609c9f7F2179b2a1f560B39F5A0",
        "gasFees": "63000",
        "gasPrice": "476190476193",
        "gasUsed": 21000,
        "toAddress": "0x73625f59CAdc5009Cb458B751b3E7b6b48C06f2C",
        "txHash": "0xa839ee83465657cac01adc1d50d96c1b586ed498120a84a64749c0034b4f19fa",
        "value": "0x"
      }
    ],
    "stateBlockNumber": 5221585,
    "totalGasUsed": 42000
  }
}
```

### flashbots_getUserStats

The `flashbots_getUserStats` JSON-RPC method returns a quick summary of how a searcher is performing in the relay,
including their [reputation-based priority](/flashbots-auction/searchers/advanced/reputation). It is currently updated
once every hour and has the following payload format:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "flashbots_getUserStats",
  "params": [
      blockNumber, //String, a hex encoded recent block number, in order to prevent replay attacks. Must be within 20 blocks of the current chain tip.
  ]
}
```

> example response:

```json
{
  "is_high_priority": true,
  "all_time_miner_payments": "1280749594841588639",
  "all_time_gas_simulated": "30049470846",
  "last_7d_miner_payments": "1280749594841588639",
  "last_7d_gas_simulated": "30049470846",
  "last_1d_miner_payments": "142305510537954293",
  "last_1d_gas_simulated": "2731770076"
}
```

where

- `is_high_priority`: boolean representing if this searcher has a high enough reputation to be in the high priority
  queue
- `all_time_miner_payments`: the total amount paid to miners over all time
- `all_time_gas_simulated`: the total amount of gas simulated across all bundles submitted to the relay. This is the
  actual gas used in simulations, not gas limit

### flashbots_getBundleStats

The `flashbots_getBundleStats` JSON-RPC method returns stats for a single bundle. You must provide a blockNumber and the
bundleHash, and the signing address must be the same as the one who submitted the bundle.

```jsonc
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "flashbots_getBundleStats",
  "params": [
    {
      bundleHash,       // String, returned by the flashbots api when calling eth_sendBundle
      blockNumber,      // String, a hex encoded block number
    }
  ]
}
```

> example response:

```jsonc
{
  "isSimulated": true,
  "isSentToMiners": true,
  "isHighPriority": true,
  "simulatedAt": "2021-08-06T21:36:06.317Z",
  "submittedAt": "2021-08-06T21:36:06.250Z",
  "sentToMinersAt": "2021-08-06T21:36:06.343Z"
}
```

