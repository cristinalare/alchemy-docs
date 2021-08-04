---
description: Frequently asked questions
---

# ❓ FAQ

## Is there an Alchemy token? <a id="is-there-an-alchemy-token"></a>

Nope! We do not currently have an Alchemy token sale but our users will be the first to know if we ever do ;\)

## How can I invest in Alchemy? <a id="how-can-i-invest-in-alchemy"></a>

Alchemy is currently a private company, if you're interested in investing, tweet out why you're interested and tag us [@AlchemyPlatform](https://twitter.com/AlchemyPlatform).

## I'm new to blockchain development, how should I get started? <a id="im-new-to-blockchain-development-how-should-i-get-started"></a>

If you are completely new to the blockchain space, you should check out our [Blockchain 101 guide](https://docs.alchemyapi.io/resources/blockchain-101) to learn more about the ins and outs of the technology behind blockchain, and the advantages of decentralization.

If you already know some blockchain stuff and are ready to dip your toes in development here are three great tutorials to check out:

1. [Simple Web3 Script](https://docs.alchemyapi.io/tutorials/simple-web3-script)
2. [Hello World Smart Contract](https://docs.alchemyapi.io/tutorials/hello-world-smart-contract)
3. [Sending Transactions Using Alchemy](https://docs.alchemyapi.io/tutorials/sending-transactions-using-web3-and-alchemy)

## Does Alchemy store my private keys? <a id="does-alchemy-store-my-private-keys"></a>

No, we will **never** store your private keys, in fact, we never even see them since we only accept _signed transactions_. This also means we cannot sign and send transactions for you. For help on how to send transactions using Alchemy, check out [this guide](https://docs.alchemyapi.io/tutorials/sending-transactions-using-web3-and-alchemy).

## What are "Already Known" errors? <a id="what-are-already-known-errors"></a>

Generally "already known" means the transaction is on the node and is in a pending state. A good solution here is to double check that your transaction nonces are correct and that there aren't any currently pending transactions in your [mempool](https://dashboard.alchemyapi.io/mempool).

## Can you change the 5s timeout:`execution aborted (timeout = 5s)`?

Unfortunately, this is a fixed timeout configured by geth so we are unable to change it. This typically occurs with expensive or heavy `eth_calls` . The best workaround is to break up your eth\_call into smaller bites to ensure they don't get timed out. 

## What happens if I run into my capacity limit? <a id="what-happens-if-i-run-into-my-capacity-limit"></a>

If you run into your capacity limit, we recommend switching on [autoscale](https://dashboard.alchemyapi.io/settings/billing) which will allow your app to continue functioning with on-demand compute.

## How can I access Archive data? <a id="how-can-i-access-archive-data"></a>

Archive data is free and available for all of our customers, no additional set up necessary!

## How do I get the timestamp for a transaction?

A transaction object will have a block number associated with it, the block number is Ethereum's measure of time, however, if you want a standard timestamp you can easily get that by making a call to [`eth_getBlockByNumber`](../documentation/apis/ethereum/#eth_getblockbynumber) and specifying the `blockNumber` field. Here is an [example request](https://composer.alchemyapi.io/?composer_state=%7B%22network%22%3A0%2C%22methodName%22%3A%22eth_getBlockByNumber%22%2C%22paramValues%22%3A%5B%22latest%22%2Cfalse%5D%7D). If you only have the transaction hash, you can get the full object by making a request to [`eth_getTransactionByHash`](../documentation/apis/ethereum/#eth_gettransactionbyhash).

## Why can't I invite a user who is already on a team? <a id="why-cant-i-invite-a-user-who-is-already-on-a-team"></a>

You can! While we don't allow a single email to be on multiple teams due to UX concerns, one thing you can do is invite the user by appending a + to their email. For example, a user with email example@gmail.com could use example+team1@gmail.com and example+team2@gmail.com to be on two different teams, while still getting all their emails to one inbox!

## Are there limits on my getLogs requests? <a id="are-there-limits-on-my-get-logs-requests"></a>

In order to ensure maximum reliability for your calls, you can make [`eth_getLogs`](https://docs.alchemyapi.io/documentation/alchemy-api-reference/json-rpc#eth_getlogs) requests with up to a _**2K block range**_ and _**no limit on the response size**_. You can also request _**any block range**_ with a cap of _**10K logs in the response**_.

If you need to pull logs frequently, we recommend [using WebSockets](https:///guides/using-websockets) to push new logs to you when they are available.

## How do I request a feature? <a id="how-do-i-request-a-feature"></a>

[Join our discord](https://discord.gg/mMGsVgd) and post in the \#feature-request channel!

## What are Alchemy's "Enhanced APIs"? <a id="what-are-alchemys-enhanced-ap-is"></a>

Glad you asked! We've built a bunch of higher level APIs to make your life as an Ethereum developer much easier. With our enhanced APIs you can access things like token balances and information, transaction history for given addresses, notifications about transaction activity, and more. Check out the full list [here](https://docs.alchemyapi.io/documentation/alchemy-api-reference).

## Where are Alchemy's servers located? <a id="where-are-alchemys-servers-located"></a>

Alchemy's servers are currently located on US East, however we serve production traffic from 99% of countries in the world and see great latency in all regions.

## Which clients does Alchemy support? <a id="which-clients-does-alchemy-support"></a>

Alchemy currently supports Geth and OpenEthereum node clients.

## How do I know the latest block is accurate? 

Great question! We've spent years developing an enhanced infrastructure and distributed node system to get the most consensus on the canonical block in the fastest amount of time. We'll only publish information if it's agreed upon by many of our nodes. 

## Do you support compressed response payloads? <a id="do-you-support-compressed-response-payloads"></a>

We do support compressed response payloads by using a Content-Encoding of gzip on all our responses.

## What are compute units? Why do you have them? <a id="what-are-compute-units-why-do-you-have-them"></a>

Compute units are a way of measuring the resources it takes to serve your traffic, we implemented this to make sure pricing is sustainable and cost-effective as you scale. Check out [this blog post](https://blog.alchemyapi.io/customer-blog-posts/alchemy-usage) for the reason we decided to implement a compute unit system.

You can see a breakdown of each method and its associated compute unit [here](https://docs.alchemyapi.io/documentation/compute-units).

## How do I distinguish between a contract address and a wallet address? <a id="how-do-i-distinguish-between-a-contract-address-and-a-wallet-address"></a>

A super easy way to distinguish between these two addresses is by calling [eth\_getCode](https:///@alchemyapi/s/alchemy/documentation/alchemy-api-reference/json-rpc#eth_getcode), which will return contract code if it's a contract and nothing if it's a wallet. Here's an example of both using our composer tool:

* [**0x Contract Address**](https://composer.alchemyapi.io/?composer_state=%7B%22network%22%3A0%2C%22methodName%22%3A%22eth_getCode%22%2C%22paramValues%22%3A%5B%220xe41d2489571d322189246dafa5ebde1f4699f498%22%2C%22latest%22%5D%7D)
* [**Vitalik's Wallet Address**](https://composer.alchemyapi.io/?composer_state=%7B%22network%22%3A0%2C%22methodName%22%3A%22eth_getCode%22%2C%22paramValues%22%3A%5B%220xAb5801a7D398351b8bE11C439e05C5B3259aeC9B%22%2C%22latest%22%5D%7D)

## What are best practices for avoiding re-orgs when calling JSON-RPC methods? <a id="how-do-i-distinguish-between-a-contract-address-and-a-wallet-address"></a>

Alchemy supports [EIP-1898](https://eips.ethereum.org/EIPS/eip-1898), which adds `blockHash` to all JSON-RPC methods that accept a default block parameter. By allowing methods with a block number parameter to also accept a block hash parameter, EIP-1898 protects against re-orgs. 

For instance, if a user executes `eth_call` for block number 10000, but the network undergoes a re-org causing the block 10000 to change, it is unclear if the call evaluated at the old block or the new one. 

{% hint style="info" %}
**Example:** `eth_getBalance`   

| Non EIP-1898 Param | EIP-1898 Param  |
| :--- | :--- |
| `["0x<some-address>", "0x<some-block-number>"]` | `["0x<some-address>", {"blockHash": "0x<some-blockhash>"}]` |
{% endhint %}

## My question isn't here, where can I get help? <a id="my-question-isnt-here-where-can-i-get-help"></a>

Don't worry, we got you. Check out our [support page](https://docs.alchemyapi.io/other/contact-us) for plenty of options!

