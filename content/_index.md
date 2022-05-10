---
title: Custom EVM RPC Methods Index
extra:
  date: 2022-05-10
  version: "0.0.3"

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


# Hardhat Network Reference

## Supported hardforks

- byzantium
- constantinople
- petersburg
- istanbul
- muirGlacier
- london
- arrowGlacier

## Config

### Supported Fields

You can set the following fields on the `networks.hardhat` config:

#### `chainId`

The chain ID number used by Hardhat Network's blockchain. Default value: `31337`.

#### `from`

The address to use as default sender. If not present the first account of the Hardhat Network is used.

#### `gas`

Its value should be `"auto"` or a number. If a number is used, it will be the gas limit used by default in every transaction. If `"auto"` is used, the gas limit will be automatically estimated. Default value: the same value as `blockGasLimit`.

Note that when using `ethers` this value will not be applied.

#### `gasPrice`

Its value should be `"auto"` or a number (in wei). This parameter behaves like `gas`. Default value: `"auto"`.

Note that when using `ethers` this value will not be applied.

#### `gasMultiplier`

A number used to multiply the results of gas estimation to give it some slack due to the uncertainty of the estimation process. Default value: `1`.

Note that when using `ethers` this value will not be applied.

#### `accounts`

This field can be configured as one of these:

- An object describing an [HD wallet]. This is the default. It can have any of the following fields:
  - `mnemonic`: a 12 or 24 word mnemonic phrases as defined by BIP39. Default value: `"test test test test test test test test test test test junk"`
  - `initialIndex`: The initial index to derive. Default value: `0`.
  - `path`: The HD parent of all the derived keys. Default value: `"m/44'/60'/0'/0"`.
  - `count`: The number of accounts to derive. Default value: `20`.
  - `accountsBalance`: string with the balance (in wei) assigned to every account derived. Default value: `"10000000000000000000000"` (10000 ETH).
  - `passphrase`: The passphrase for the wallet. Default value: empty string.
- An array of the initial accounts that the Hardhat Network will create. Each of them must be an object with `privateKey` and `balance` fields.

#### `blockGasLimit`

The block gas limit to use in Hardhat Network's blockchain. Default value: `30_000_000`

#### `hardfork`

This setting changes how Hardhat Network works, to mimic Ethereum's mainnet at a given hardfork. It must be one of `"byzantium"`, `"constantinople"`, `"petersburg"`, `"istanbul"`, `"muirGlacier"`, `"berlin"`, `"london"` and `"arrowGlacier"`. Default value: `"arrowGlacier"`

#### `throwOnTransactionFailures`

A boolean that controls if Hardhat Network throws on transaction failures. If this value is `true`, Hardhat Network will throw [combined JavaScript and Solidity stack traces](../README.md#solidity-stack-traces) on transaction failures. If it is `false`, it will return the failing transaction hash. In both cases the transactions are added into the blockchain. Default value: `true`

#### `throwOnCallFailures`

A boolean that controls if Hardhat Network throws on call failures. If this value is `true`, Hardhat Network will throw [combined JavaScript and Solidity stack traces](../README.md#solidity-stack-traces) when a call fails. If it is `false`, it will return the call's `return data`, which can contain a revert reason. Default value: `true`

#### `loggingEnabled`

A boolean that controls if Hardhat Network logs every request or not. Default value: `false` for the in-process Hardhat Network provider, `true` for the Hardhat Network backed JSON-RPC server (i.e. the `node` task).

#### `initialDate`

An optional string setting the date of the blockchain. Valid values are [Javascript's date time strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse#Date_Time_String_Format). Default value: the current date and time if not forking another network. When forking another network, the timestamp of the block you forked from, plus one second, is used.

#### `allowUnlimitedContractSize`

An optional boolean that disables the contract size limit imposed by the [EIP 170](https://eips.ethereum.org/EIPS/eip-170). Default value: `false`

#### `forking`

An object that describes the [forking](../guides/mainnet-forking.md) configuration that can have the following fields:

- `url`: a URL that points to a JSON-RPC node with state that you want to fork off. There's no default value for this field. It must be provided for the fork to work.
- `blockNumber`: an optional number to pin which block to fork from. If no value is provided, the latest block is used.
- `enabled`: an optional boolean to switch on or off the fork functionality. Default value: `true` if `url` is set, `false` otherwise.

#### `chains`

An object that configures chain-specific options. Each key is a number representing a chain ID, and each value is an object configuring the chain with that ID. In the inner object, the following fields are supported:

- `hardforkHistory`: an object whose keys are strings representing hardfork names (eg `"london"`, `"berlin"`) and whose values are numbers specifying the block at which that hardfork was activated.

The default value includes configurations for several well known chains (eg mainnet, chain ID `1`); using this field is only useful when forking unusual networks. The user may override the defaults for some chain ID's while leaving the defaults in place for other chain ID's. Overriding the default for a chain ID will replace the entire configuration for that chain.

For more details, see [Using a custom hardfork history](../guides/mainnet-forking.md#using-a-custom-hardfork-history).

#### `minGasPrice`

The minimum `gasPrice` that a transaction must have. This field must not be present if the `"hardfork"` is `"london"` or a later one. Default value: `"0"`.

#### `initialBaseFeePerGas`

The `baseFeePerGas` of the first block. Note that when forking a remote network, the "first block" is the one immediately after the block you forked from. This field must not be present if the `"hardfork"` is not `"london"` or a later one. Default value: `"1000000000"` if not forking. When forking a remote network, if the remote network uses EIP-1559, the first local block will use the right `baseFeePerGas` according to the EIP, otherwise `"10000000000"` is used.

- `coinbase`: The address used as coinbase in new blocks. Default value: `"0xc014ba5ec014ba5ec014ba5ec014ba5ec014ba5e"`.

### Mining modes

You can configure the mining behavior under your Hardhat Network settings:

```js
networks: {
  hardhat: {
    mining: {
      auto: false,
      interval: 5000
    }
  }
}
```

In this example, automining is disabled and interval mining is set so that a new block is generated every 5 seconds. You can also configure interval mining to generate a new block after a random delay:

```js
networks: {
  hardhat: {
    mining: {
      auto: false,
      interval: [3000, 6000]
    }
  }
}
```

In this case, a new block will be mined after a random delay of between 3 and 6 seconds. For example, the first block could be mined after 4 seconds, the second block 5.5 seconds after that, and so on.

See also [Mining Modes](../explanation/mining-modes.md).

#### Manual mining

You can disable both mining modes like this:

```js
networks: {
  hardhat: {
    mining: {
      auto: false,
      interval: 0
    }
  }
}
```

This means that no new blocks will be mined by the Hardhat Network, but you can manually mine new blocks using the `evm_mine` RPC method. This will generate a new block that will include as many pending transactions as possible.

#### Transaction ordering

Hardhat Network can sort mempool transactions in two different ways. How they are sorted will alter which transactions from the mempool get included in the next block, and in which order.

The first ordering mode, called `"priority"`, mimics Geth's behavior. This means that it prioritizes transactions based on the fees paid to the miner. This is the default.

The second ordering mode, called `"fifo"`, keeps the mempool transactions sorted in the order they arrive.

You can change the ordering mode with:

```json
networks: {
  hardhat: {
    mining: {
      mempool: {
        order: "fifo"
      }
    }
  }
}
```

## `console.log`

- You can use it in calls and transactions. It works with `view` functions, but not in `pure` ones.
- It always works, regardless of the call or transaction failing or being successful.
- To use it you need to import `hardhat/console.sol`.
- You can call `console.log` with up to 4 parameters in any order of following types:
  - `uint`
  - `string`
  - `bool`
  - `address`
- There's also the single parameter API for the types above, and additionally `bytes`, `bytes1`... up to `bytes32`:
  - `console.logInt(int i)`
  - `console.logUint(uint i)`
  - `console.logString(string memory s)`
  - `console.logBool(bool b)`
  - `console.logAddress(address a)`
  - `console.logBytes(bytes memory b)`
  - `console.logBytes1(bytes1 b)`
  - `console.logBytes2(bytes2 b)`
  - ...
  - `console.logBytes32(bytes32 b)`
- `console.log` implements the same formatting options that can be found in Node.js' [`console.log`](https://nodejs.org/dist/latest-v12.x/docs/api/console.html#console_console_log_data_args), which in turn uses [`util.format`](https://nodejs.org/dist/latest-v12.x/docs/api/util.html#util_util_format_format_args).
  - Example: `console.log("Changing owner from %s to %s", currentOwner, newOwner)`
- `console.log` is implemented in standard Solidity and then detected in Hardhat Network. This makes its compilation work with any other tools (like Remix, Waffle or Truffle).
- `console.log` calls can run in other networks, like mainnet, kovan, ropsten, etc. They do nothing in those networks, but do spend a minimal amount of gas.
- `console.log` output can also be viewed for testnets and mainnet via [Tenderly](https://tenderly.co/).
- `console.log` works by sending static calls to a well-known contract address. At runtime, Hardhat Network detects calls to that address, decodes the input data to the calls, and writes it to the console.

## Initial State

Hardhat Network is initialized by default in this state:

- A brand new blockchain, just with the genesis block.
- 20 accounts with 10000 ETH each, generated with the mnemonic `"test test test test test test test test test test test junk"`. Their addresses are:
  - `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`
  - `0x70997970C51812dc3A010C7d01b50e0d17dc79C8`
  - `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC`
  - `0x90F79bf6EB2c4f870365E785982E1f101E93b906`
  - `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65`
  - `0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc`
  - `0x976EA74026E726554dB657fA54763abd0C3a0aa9`
  - `0x14dC79964da2C08b23698B3D3cc7Ca32193d9955`
  - `0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f`
  - `0xa0Ee7A142d267C1f36714E4a8F75612F20a79720`
  - `0xBcd4042DE499D14e55001CcbB24a551F3b954096`
  - `0x71bE63f3384f5fb98995898A86B02Fb2426c5788`
  - `0xFABB0ac9d68B0B445fB7357272Ff202C5651694a`
  - `0x1CBd3b2770909D4e10f157cABC84C7264073C9Ec`
  - `0xdF3e18d64BC6A983f673Ab319CCaE4f1a57C7097`
  - `0xcd3B766CCDd6AE721141F452C550Ca635964ce71`
  - `0x2546BcD3c84621e976D8185a91A922aE77ECEc30`
  - `0xbDA5747bFD65F08deb54cb465eB87D40e51B197E`
  - `0xdD2FD4581271e230360230F9337D5c0430Bf44C0`
  - `0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199`

To customise it, take a look at [the configuration section](/config/README.md#hardhat-network).

## JSON-RPC methods support

### Standard methods

#### `debug_traceTransaction`

Get debug traces of already-mined transactions.

To get a trace, call this method with the hash of the transaction as its argument:

```js
const trace = await hre.network.provider.send("debug_traceTransaction", [
  "0x123...",
]);
```

You can also selectively disable some properties in the list of steps:

```js
const trace = await hre.network.provider.send("debug_traceTransaction", [
  "0x123...",
  {
    disableMemory: true,
    disableStack: true,
    disableStorage: true,
  },
]);
```


### Hardhat network methods

#### `hardhat_addCompilationResult`

Add information about compiled contracts

#### `hardhat_dropTransaction`

Remove a transaction from the mempool

<!-- intentionally undocumented, internal method:
#### `hardhat_getStackTraceFailuresCount`
-->

#### `hardhat_impersonateAccount`

Hardhat Network allows you to send transactions impersonating specific account and contract addresses.

To impersonate an account use the this method, passing the address to impersonate as its parameter:

```tsx
await hre.network.provider.request({
  method: "hardhat_impersonateAccount",
  params: ["0x364d6D0333432C3Ac016Ca832fb8594A8cE43Ca6"],
});
```

If you are using [`hardhat-ethers`](https://github.com/nomiclabs/hardhat/tree/master/packages/hardhat-ethers), call `getSigner` after impersonating the account:

```
const signer = await ethers.getSigner("0x364d6D0333432C3Ac016Ca832fb8594A8cE43Ca6")
signer.sendTransaction(...)
```

Call [`hardhat_stopImpersonatingAccount`](#hardhat-stopimpersonatingaccount) to stop impersonating.

<!-- intentionally undocumented, internal method:
#### `hardhat_intervalMine`
-->

#### `hardhat_getAutomine`

Returns `true` if automatic mining is enabled, and `false` otherwise. See [Mining Modes](../explanation/mining-modes.md) to learn more.

#### `hardhat_mine`

Sometimes you may want to advance the latest block number of the Hardhat Network by a large number of blocks. One way to do this would be to call the `evm_mine` RPC method multiple times, but this is too slow if you want to mine thousands of blocks. The `hardhat_mine` method can mine any number of blocks at once, in constant time. (It exhibits the same performance no matter how many blocks are mined.)

`hardhat_mine` accepts two parameters, both of which are optional. The first parameter is the number of blocks to mine, and defaults to 1. The second parameter is the interval between the timestamps of each block, _in seconds_, and it also defaults to 1. (The interval is applied only to blocks mined in the given method invocation, not to blocks mined afterwards.)

```js
// mine 256 blocks
await hre.network.provider.send("hardhat_mine", ["0x100"]);

// mine 1000 blocks with an interval of 1 minute
await hre.network.provider.send("hardhat_mine", ["0x3e8", "0x3c"]);
```

Note that most blocks mined via this method (all except for the final one) may not technically be valid blocks. Specifically, they have an invalid parent hash, the coinbase account will not have been credited with block rewards, and the `baseFeePerGas` will be incorrect. (The final block in a sequence produced by `hardhat_mine` will always be fully valid.)

Also note that blocks created via `hardhat_mine` may not trigger new-block events, such as filters created via `eth_newBlockFilter` and WebSocket subscriptions to new-block events.

#### `hardhat_reset`

See the [Mainnet Forking guide](../guides/mainnet-forking.md)

#### `hardhat_setBalance`

Modifies the balance of an account.

For example:

```tsx
await network.provider.send("hardhat_setBalance", [
  "0x0d2026b3EE6eC71FC6746ADb6311F6d3Ba1C000B",
  "0x1000",
]);
```

This will result in account `0x0d20...000B` having a balance of 4096 wei.

#### `hardhat_setCode`

Modifies the bytecode stored at an account's address.

For example:

```tsx
await network.provider.send("hardhat_setCode", [
  "0x0d2026b3EE6eC71FC6746ADb6311F6d3Ba1C000B",
  "0xa1a2a3...",
]);
```

This will result in account `0x0d20...000B` becoming a smart contract with bytecode `a1a2a3....` If that address was already a smart contract, then its code will be replaced by the specified one.

#### `hardhat_setCoinbase`

Sets the coinbase address to be used in new blocks.

For example:

```tsx
await network.provider.send("hardhat_setCoinbase", [
  "0x0d2026b3EE6eC71FC6746ADb6311F6d3Ba1C000B",
]);
```

This will result in account `0x0d20...000B` being used as miner/coinbase in every new block.

#### `hardhat_setLoggingEnabled`

Enable or disable logging in Hardhat Network

#### `hardhat_setMinGasPrice`

Change the minimum gas price accepted by the network (in wei)

#### `hardhat_setNextBlockBaseFeePerGas`

Sets the base fee of the next block.

For example:

```tsx
await network.provider.send("hardhat_setNextBlockBaseFeePerGas", [
  "0x2540be400", // 10 gwei
]);
```

This only affects the next block; the base fee will keep being updated in each subsequent block according to [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559).

#### `hardhat_setNonce`

Modifies an account's nonce by overwriting it.

For example:

```typescript
await network.provider.send("hardhat_setNonce", [
  "0x0d2026b3EE6eC71FC6746ADb6311F6d3Ba1C000B",
  "0x21",
]);
```

This will result in account `0x0d20...000B` having a nonce of 33.

Throws an `InvalidInputError` if nonce is smaller than the current one. The reason for this restriction is to avoid collisions when deploying contracts using the same nonce more than once.

You can only use this method to increase the nonce of an account; you can't set a lower value than the account's current nonce.

#### `hardhat_setStorageAt`

Writes a single position of an account's storage.

For example:

```typescript
await network.provider.send("hardhat_setStorageAt", [
  "0x0d2026b3EE6eC71FC6746ADb6311F6d3Ba1C000B",
  "0x0",
  "0x0000000000000000000000000000000000000000000000000000000000000001",
]);
```

This will set the contract's first storage position (at index `0x0`) to 1.

The mapping between a smart contract's variables and its storage position is not straightforward except in some very simple cases. For example, if you deploy this contract:

```solidity
contract Foo {
  uint public x;
}
```

And you set the first storage position to 1 (as shown in the previous snippet), then calling `foo.x()` will return 1.

The storage position index must not exceed 2^256, and the value to write must be exactly 32 bytes long.

#### `hardhat_stopImpersonatingAccount`

Use this method to stop impersonating an account after having previously used [`hardhat_impersonateAccount`](#hardhat-impersonateaccount), like:

```tsx
await hre.network.provider.request({
  method: "hardhat_stopImpersonatingAccount",
  params: ["0x364d6D0333432C3Ac016Ca832fb8594A8cE43Ca6"],
});
```

### Special testing/debugging methods

#### `evm_increaseTime`

Same as Ganache.

#### `evm_mine`

Same as Ganache

#### `evm_revert`

Same as Ganache.

#### `evm_setAutomine`

Enables or disables, based on the single boolean argument, the automatic mining of new blocks with each new transaction submitted to the network. You can use [`hardhat_getAutomine`](#hardhat-getautomine) to get the current value. See also [Mining Modes](../explanation/mining-modes.md).

#### `evm_setBlockGasLimit`

#### `evm_setIntervalMining`

Enables (with a numeric argument greater than 0) or disables (with a numeric argument equal to 0), the automatic mining of blocks at a regular interval of milliseconds, each of which will include all pending transactions. See also [Mining Modes](../explanation/mining-modes.md).

#### `evm_setNextBlockTimestamp`

This method works like `evm_increaseTime`, but takes the exact timestamp that you want in the next block, and increases the time accordingly.

#### `evm_snapshot`

Same as [Ganache](https://github.com/trufflesuite/ganache/blob/ef1858d5d6f27e4baeb75cccd57fb3dc77a45ae8/src/chains/ethereum/ethereum/RPC-METHODS.md#evm_snapshot).

Snapshot the state of the blockchain at the current block. Takes no parameters. Returns the id of the snapshot that was created. A snapshot can only be reverted once. After a successful `evm_revert`, the same snapshot id cannot be used again. Consider creating a new snapshot after each `evm_revert` if you need to revert to the same point multiple times.