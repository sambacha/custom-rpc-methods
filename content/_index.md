---
title: Custom EVM RPC Methods Index
extra:
  date: 2022-05-05
  version: "0.0.1"

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
    

