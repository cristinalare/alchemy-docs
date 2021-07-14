---
description: >-
  Alchemy offers several enhanced API methods on top of the existing web3.js
  calls.
---

# Enhanced Web3 API

#### _**Get access to**_ [_**Alchemy for free here**_](https://alchemy.com/?r=e68b2f77-7fc7-4ef7-8e9c-cdfea869b9b5)_**.**_

To learn more about the standard web3.js API calls, check out their documentation [here](https://web3js.readthedocs.io/en/v1.2.9/). 

## 😎 Enhanced API Calls

### web3.alchemy.getTokenAllowance\({contract, owner, spender}\)

Returns token balances for a specific address given a list of contracts.

#### **Parameters:**

An object with the following fields:

* `contract`: The address of the token contract.
* `owner`: The address of the token owner.
* `spender`: The address of the token spender.

#### **Returns:**

The allowance amount, as a string representing a base-10 number.

### web3.alchemy.getTokenBalances\(address, contractAddresses\)

Returns token balances for a specific address given a list of contracts.

#### **Parameters:**

1. `address`: The address for which token balances will be checked.
2. `contractAddresses`: An array of contract addresses.

#### **Returns:**

An object with the following fields:

* `address`: The address for which token balances were checked.
* `tokenBalances`: An array of token balance objects. Each object contains:
  * `contractAddress`: The address of the contract.
  * `tokenBalance`: The balance of the contract, as a string representing a base-10 number.
  * `error`: An error string. One of this or `tokenBalance` will be `null`.

### web3.alchemy.getTokenMetadata\(address\)

Returns metadata \(name, symbol, decimals, logo\) for a given token contract address.

#### **Parameters:**

`address`: The address of the token contract.

#### **Returns:**

An object with the following fields:

* `name`: The token's name. `null` if not defined in the contract and not available from other sources.
* `symbol`: The token's symbol. `null` if not defined in the contract and not available from other sources.
* `decimals`: The token's decimals. `null` if not defined in the contract and not available from other sources.
* `logo`: URL of the token's logo image. `null` if not available.

### web3.eth.subscribe\("alchemy\_fullPendingTransactions"\)

Subscribes to pending transactions, similar to the standard Web3 call `web3.eth.subscribe("pendingTransactions")`, but differs in that it emits full transaction information rather than just transaction hashes.

Note that the argument passed to this function is `"alchemy_fullPendingTransactions"`, which is different from the string used in raw [`eth_subscribe`](../apis/ethereum.md#eth_subscribe) JSON-RPC calls, where it is `"alchemy_newFullPendingTransactions"` instead. This is confusing, but it is also consistent with the existing Web3 subscription APIs \(for example: `web3.eth.subscribe("pendingTransactions")` vs `"newPendingTransactions"` in raw JSON-RPC\).

### web3.alchemy.getAssetTransfers\({fromBlock, toBlock, fromAddress, toAddress, contractAddresses, excludeZeroValue, maxCount, category, pageKey}\)

Returns an array of asset transfers based on the specified paramaters.

#### **Parameters:**

* Parameters:
  * Object - An object with the following fields \(required\):
    * `fromBlock`: in hex string or "latest". optional \(default to latest\)
    * `toBlock`: in hex string or "latest". optional \(default to latest\)
    * `fromAddress`: in hex string. optional
    * `toAddress`: in hex string. optional.
    * `contractAddresses`: list of hex strings. optional.
    * `category`: list of any combination of `external`, `token`. optional, if blank, would include both.
    * `excludeZeroValue:` a`Boolean` . optional \(default `true`\)
    * `maxCount`: max number of results to return per call. optional \(default `1000)`
    * `pageKey`: for [pagination](../apis/enhanced-apis/transfers-api.md#pagination). optional
* `fromBlock` and `toBlock` are inclusive. Both default to `latest` if not specified.
* `fromAddress` and `toAddress` will be `AND`ed together when filtering. If left blank, will indicate a wildcard \(any address\).
* `contractAddresses` only applies to `token` category transfers \(eth log events\). The list of addresses are `OR`ed together. This filter will be `AND`ed with `fromAddress` and `toAddress` for eth log events. If empty, or unspecified, it will be taken as a wildcard \(any contract addresses\).
* `category`: `external` for primary level eth transfers, `token` for contract event transfers.
* `excludeZeroValue:` an optional `Boolean` to exclude asset transfers with a value field of zero \(defaults to `true`\)
* `maxCount`: The maximum number of results to return per call. Default and max will be 1000.
* `pageKey`: If left blank, will return the first 1000 or `maxCount` number of results. If more results are available, a uuid pageKey will be returned in the response. Pass that uuid into `pageKey` to fetch the next 1000 or maxCount. See section on [pagination](../apis/enhanced-apis/transfers-api.md#pagination). 

  


