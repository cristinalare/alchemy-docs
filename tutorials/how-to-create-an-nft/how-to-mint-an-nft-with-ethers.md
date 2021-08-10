---
description: >-
  This tutorial describes how to mint an NFT on the Ethereum blockchain using
  Ethers via the ethers.js library, and our smart contract from Part I: How to Create an NFT.
  We'll also explore basic test setup using dependency injection.
---

# 🪄How to Mint an NFT Using Ethers.js

_Estimated time to complete this guide: ~10 minutes_ 

In [another tutorial](how-to-mint-an-nft-with-ethers), we learned how to mint an NFT using Web3 and the [OpenZeppelin contracts
library](https://docs.openzeppelin.com/contracts/erc721). In this exercise, we're going to walk you through an alternative implementation using version 4 of the [OpenZeppelin library](https://docs.openzeppelin.com/contracts/4.x/erc721) as well as the [Ethers.js](https://docs.ethers.io/) Ethereum library instead of Web3.

We'll also cover the basics of testing your contract with [Hardhat and Waffle](https://hardhat.org/plugins/nomiclabs-hardhat-waffle.html). For this tutorial I'm using Yarn, but you can use npm/npx if you prefer.

Lastly, we'll use TypeScript.

In all other respects, this tutorial works the same as the Web3 version, including tools such as Pinata and IPFS.

## A Quick Reminder

As a reminder, "minting an NFT" is the act of publishing a unique instance of your ERC721 token on the blockchain.
This tutorial assumes that that you've successfully [deployed a smart contract to the Ropsten network in Part I](how-to-mint-a-nft)
of the NFT tutorial series, which includes [installing Ethers](../how-to-create-an-nft#step-12-install-ethers-js).

### Step 1: Create your Solidity contract <a id="step-1-create-your-solidity-contract"></a>

OpenZeppelin is library for secure smart contract development. You simply inherit their implementations of popular standards such as ERC20 or ERC721, and extend the behavior to your needs. We're going to put this file at `contracts/MyNFT.sol`.

```solidity
// Contract based on https://docs.openzeppelin.com/contracts/4.x/erc721
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

### Step 2: Create a mint-nft.ts file <a id="step-2-create-a-mint-nft-ts-file"></a>

Create a `mint-nft.ts` file mediate our interaction with the Solidity contract via Ethers.js. I put this under a `lib` directory, but you can organize however you prefer.


```ts
import { Contract, ethers } from "ethers";
import contractArtifact from "../artifacts/contracts/MyNFT.sol/MyNFT.json";
import { FactoryOptions } from "@nomiclabs/hardhat-ethers/src/types/index";
import { safeEnv } from "./safe-env";
import { TransactionResponse } from "@ethersproject/abstract-provider";

export function mintNft(
  tokenURI: string,
  env: safeEnv,
  provider: ethers.providers.Provider
): Promise<TransactionResponse> {
  const NFT_CONTRACT = getContract(env, provider);
  return NFT_CONTRACT.mintNFT(env("ETH_PUBLIC_KEY"), tokenURI, {
    gasLimit: 500_000,
  });
}

export function mintNftAndPrintHash(
  tokenURI: string,
  env: safeEnv,
  provider: ethers.providers.Provider
): Promise<void> {
  return mintNft(tokenURI, env, provider).then((tr: TransactionResponse) => {
    process.stdout.write(`TX hash: ${tr.hash}`);
  });
}

function getContract(
  env: safeEnv,
  provider: ethers.providers.Provider
): Contract {
  const WALLET = new ethers.Wallet(env("ETH_PRIVATE_KEY"), provider);

  return new ethers.Contract(
    env("NFT_CONTRACT_ADDRESS"),
    contractArtifact.abi,
    WALLET
  );
}

type contractFactoryGetter = (
  name: string,
  signerOrOptions?: ethers.Signer | FactoryOptions
) => Promise<ethers.ContractFactory>;

export async function deploy(
  env: safeEnv,
  provider: ethers.providers.Provider,
  cfg: contractFactoryGetter
): Promise<Contract> {
  const WALLET = new ethers.Wallet(env("ETH_PRIVATE_KEY"), provider);

  return cfg("MyNFT", WALLET).then((contract: ethers.ContractFactory) =>
    contract.deploy()
  );
}

export function getProvider(): ethers.providers.Provider {
  return ethers.getDefaultProvider("ropsten", {
    alchemy: process.env.ALCHEMY_API_KEY,
  });
}
```

Note that the final `getProvider()` function uses the ropsten network. This argument is optional and defaults to "homestead" if omitted. We're using Alchemy of course, but there are several [supported alternatives](https://docs.ethers.io/v5/api/providers/#providers-getDefaultProvider).

### Step 3: Create Hardhat tasks to deploy our contract and mint NFT's <a id="step-3-create-hardhat-tasks"></a>

Create the file `tasks/nft.ts` containing the following:

```ts
import { deploy, getProvider, mintNftAndPrintHash } from "../lib/mint-nft";
import { task, types } from "hardhat/config";
import { processEnv } from "../lib/safe-env";

task("deploy-nft", "Deploy NFT contract").setAction(
  async (taskArgs, { ethers: { getContractFactory } }) => {
    return deploy(processEnv, getProvider(), getContractFactory).then(
      (result) => process.stdout.write(`Contract address: ${result.address}`)
    );
  }
);

task("mint-nft", "Mint an NFT")
  .addParam("tokenUri", "Your ERC721 Token URI", undefined, types.string)
  .setAction(async (tokenUri) => {
    const provider = getProvider();
    return mintNftAndPrintHash(tokenUri, processEnv, provider);
  });
```

### Step 4: Create unit tests<a id="step-4-create-unit-tests"></a>

Create a `test/MyNFT.spec.ts` file containing the following code. This will test the contract itself. Note that we're leaving most of [methods inherited from the OpenZeppelin library](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721) untested for the sake of brevity. These tests should provide a firm foundation for you to test as much of the functionality as you want.

```ts
import { ethers, waffle } from "hardhat";
import { Contract, Wallet } from "ethers";
import { expect } from "chai";
import { deploy } from "../lib/mint-nft";
import { newProxyEnv } from "../lib/safe-env";
import { TransactionResponse } from "@ethersproject/abstract-provider";

describe("MyNFT", () => {
  const TOKEN_URI = "http://example.com/ip_records/42";
  let deployedContract: Contract;
  let wallet: Wallet;

  beforeEach(async () => {
    [wallet] = waffle.provider.getWallets(); // https://hardhat.org/hardhat-network/reference/#initial-state
    deployedContract = await deploy(
      newProxyEnv({
        ETH_PRIVATE_KEY: wallet.privateKey,
      }),
      waffle.provider,
      ethers.getContractFactory
    );
  });

  async function mintNftDefault(): Promise<TransactionResponse> {
    return deployedContract.mintNFT(wallet.address, TOKEN_URI);
  }

  describe("mintNft", async () => {
    it("emits the Transfer event", () => {
      return expect(mintNftDefault())
        .to.emit(deployedContract, "Transfer")
        .withArgs(ethers.constants.AddressZero, wallet.address, "1");
    });

    it("returns the new item ID", async () => {
      expect(
        await deployedContract.callStatic.mintNFT(wallet.address, TOKEN_URI)
      ).to.eq("1");
    });

    it("increments the item ID", async () => {
      const STARTING_NEW_ITEM_ID = "1";
      const NEXT_NEW_ITEM_ID = "2";

      await expect(mintNftDefault())
        .to.emit(deployedContract, "Transfer")
        .withArgs(
          ethers.constants.AddressZero,
          wallet.address,
          STARTING_NEW_ITEM_ID
        );

      await expect(mintNftDefault())
        .to.emit(deployedContract, "Transfer")
        .withArgs(
          ethers.constants.AddressZero,
          wallet.address,
          NEXT_NEW_ITEM_ID
        );
    });

    it("cannot mint to address zero", () => {
      const TX = deployedContract.mintNFT(
        ethers.constants.AddressZero,
        TOKEN_URI
      );
      return expect(TX).to.be.revertedWith("ERC721: mint to the zero address");
    });
  });

  describe("balanceOf", () => {
    it("gets the count of NFTs for this address", async () => {
      await expect(await deployedContract.balanceOf(wallet.address)).to.eq("0");

      await mintNftDefault();

      return expect(await deployedContract.balanceOf(wallet.address)).to.eq(
        "1"
      );
    });
  });
});
```

### Step 5: Create integration tests <a id="step-5-create-integration-tests"></a>

Create a `test/lib/mint-nft.spec.ts` file containing the following. This will test the combined behavior of our contract as well as its interaction with our helper functions.

```ts
import { ethers, waffle } from "hardhat";
import { Contract, Wallet } from "ethers";
import { expect } from "chai";
import { deploy, mintNft } from "../../lib/mint-nft";
import { safeEnv, newProxyEnv } from "../../lib/safe-env";

describe("deploying and minting NFT's", () => {
  const TOKEN_URI = "http://example.com/ip_records/42";
  let deployedContract: Contract;
  let wallet: Wallet;
  let env: safeEnv;

  beforeEach(async () => {
    [wallet] = waffle.provider.getWallets();
    const envProperties: NodeJS.ProcessEnv = {
      ETH_PUBLIC_KEY: wallet.address,
      ETH_PRIVATE_KEY: wallet.privateKey,
    };
    env = newProxyEnv(envProperties);
    deployedContract = await deploy(
      env,
      waffle.provider,
      ethers.getContractFactory
    );
    envProperties["NFT_CONTRACT_ADDRESS"] = deployedContract.address;
  });

  it("calls through and returns the transaction object", () => {
    return expect(mintNft(TOKEN_URI, env, waffle.provider))
      .to.emit(deployedContract, "Transfer")
      .withArgs(ethers.constants.AddressZero, wallet.address, "1");
  });
});
```

## Running Our Tasks

To see our tasks, we run `hardhat` and see this output (excluding the built-in tasks).

```zsh
AVAILABLE TASKS:

  deploy-nft    Deploy NFT contract
  mint-nft      Mint an NFT
```

Forget the arguments to your task? No problem.

```zsh
$ hardhat help deploy-nft

Usage: hardhat [GLOBAL OPTIONS] deploy-nft

deploy-nft: Deploy NFT contract
```

## Running Our Tests

To run our tests, we run `hardhat test`.

```zsh
  mintNft
    ✓ calls through and returns the transaction object (60ms)

  MyNFT
    mintNft
      ✓ emits the Transfer event (60ms)
      ✓ returns the new item ID
      ✓ increments the item ID (57ms)
      ✓ cannot mint to address zero
    balanceOf
      ✓ gets the count of NFTs for this address


  6 passing (2s)

✨  Done in 5.66s.
```

## Putting It All Together

What we've done is to cram as much of our deploying and minting code as possible into a library of functions that we can thoroughly test via [dependency injection](https://wiki.c2.com/?DependencyInjection). Our Hardhat tasks then merely pipe our dependencies (command line arguments and environment variables) as parameters into those functions, minimizing the footprint of any untested code. The wallet provided by `waffle.provider.getWallets()` links to a [Hardhat Network](https://hardhat.org/hardhat-network/) account that [conveniently comes preloaded](https://hardhat.org/hardhat-network/reference/#initial-state) with an eth balance that we can use to fund our test transactions.
