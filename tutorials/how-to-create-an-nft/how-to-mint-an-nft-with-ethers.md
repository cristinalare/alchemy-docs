---
description: >-
  This tutorial describes how to mint an NFT on the Ethereum blockchain using
  Ethers via the ethers.js library, and our smart contract from Part I: How to Create an NFT.
  We'll also explore basic test setup using dependency injection.
---

# 🪄How to Mint an NFT Using Ethers.js

_Estimated time to complete this guide: ~10 minutes_ 

In (another tutorial)[how-to-mint-an-nft-with-ethers], we learned how to mint an NFT using Web 3 and the OpenZeppelin contracts
library version 3.1.0. In this exercise, we're going to walk you through an alternative implementation based on the newer OpenZeppelin contracts version 4.2.0,
as well as the [Ethers.js](https://docs.ethers.io/) library instead of Web 3. We'll also cover the basics of testing your contract with
[Hardhat and Waffle](https://hardhat.org/plugins/nomiclabs-hardhat-waffle.html). For this tutorial I'm using Yarn, but you can use npm if you prefer.
Lastly, we'll use TypeScript.

In all other respects, this tutorial works the same as the Web 3 version, including tools such as Pinata and IPFS. The final step of running your script on the
command line remains unchanged.

## A Quick Reminder

As a reminder, "Minting an NFT" is the act of publishing a unique instance of your ERC721 token on the blockchain.
This tutorial assumes that that you've successfully [deployed a smart contract to the Ropsten network in Part I](how-to-write-and-deploy-a-nft-smart-contract)
of the NFT tutorial series, which includes [installing Ethers](./how-to-create-an-nft#step-12-install-ethers-js). However, you'll want to update your MyNFT.sol to support the latest OpenZeppelin library.

```solidity
// Contract based on https://docs.openzeppelin.com/contracts/3.x/erc721
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract MyNFT is ERC721URIStorage {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() ERC721("MyNFT", "MNFT") {}

    function mintNFT(address recipient, string memory tokenURI)
    public
    returns (uint256)
    {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(recipient, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
}
```

### Step 1: Create a mint-nft.ts file <a id="step-1-create-a-mint-nft-ts-file"></a>

Create a `mint-nft.ts` file (I put this under a lib directory) and add the following lines of code:

```ts
import {ethers} from 'ethers';

const contractArtifact = require("../artifacts/contracts/MyNFT.sol/MyNFT.json");

export async function mintNft(
    tokenURI: string,
    env: NodeJS.ProcessEnv,
    provider: ethers.providers.Provider,
) {
    const publicKey = env.PUBLIC_KEY!;
    const privateKey = env.PRIVATE_KEY!;
    const wallet = new ethers.Wallet(privateKey, provider);

    const nftContract = new ethers.Contract(env.CONTRACT_ADDRESS!, contractArtifact.abi, wallet);

    return nftContract.mintNFT(publicKey, tokenURI, { gasLimit: 500_000})
}
```

### Step 2: Create a scripts/mint-nft.ts <a id="step-2-create-a-script-mint-nft-ts-file"></a>

Create a `scripts/mint-nft.ts` file and add the following lines of code:

```ts
const hre = require("hardhat");
import { mintNft } from "../lib/mint-nft";
import { ethers } from "hardhat";

async function main() {
  // E.g. if your URL is:
  // https://eth-ropsten.alchemyapi.io/v2/8-bL_8ufTHsTfH-DLyOoDbsM1epi0RcC
  // then ALCHEMY_API_KEY is 8-bL_8ufTHsTfH-DLyOoDbsM1epi0RcC
  const provider = ethers.getDefaultProvider("ropsten", {
    alchemy: process.env.ALCHEMY_API_KEY!,
  });

  // The token URI value will ultimately be passed from our web application and parsed via
  // yargs, but we won't go into that here.
  const tokenURI = "http://example.com/your-endpoint";
  mintNft(tokenURI, process.env, provider);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### Step 3: Create a test/mint-nft.spec.ts <a id="step-3-create-a-test-mint-nft-spec-ts-file"></a>

Create a `test/mint-nft.spec.ts` file and add the following lines of code:

```ts
import {ethers, waffle} from "hardhat";
import { Contract } from "@ethersproject/contracts";
import chai, { expect } from "chai";
import { solidity } from "ethereum-waffle";
import { mintNft } from "../lib/mint-nft"

chai.use(solidity);

describe("MyNFT", function () {
  let signerAddress: string;
  const tokenURI = "http://example.com/ip_records/42";
  let env: NodeJS.ProcessEnv;
  let deployedContract: Contract;

  beforeEach(async () => {
    const [wallet] = waffle.provider.getWallets();
    const MyNft = await ethers.getContractFactory("MyNFT", wallet);
    const hardhatMyNft = await MyNft.deploy();
    deployedContract = await hardhatMyNft.deployed();

    env = {
      CONTRACT_ADDRESS: deployedContract.address,
      PUBLIC_KEY: wallet.address,
      PRIVATE_KEY: wallet.privateKey
    }
  })

  it("works", async function () {
    await expect(mintNft(tokenURI, env, ethers.provider))
      .to.emit(deployedContract, 'Transfer');
  });
});
```

What we've done here is to create a library function which contains as much of the behavior as possible so that we can maximize test coverage,
using [dependency injection](https://wiki.c2.com/?DependencyInjection). Our script therefore contains no logic, and merely injects the necessary
parameters into our minting function. The wallet provided by `waffle.provider.getWallets()` links to a [Hardhat Network](https://hardhat.org/hardhat-network/) account that conveniently comes preloaded with an eth balance that we can use to fund our test transactions.
