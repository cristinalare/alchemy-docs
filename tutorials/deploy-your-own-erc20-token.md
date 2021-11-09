---
description: Tutorial for deploying your own ERC-20 token on Rinkeby
---

# 🪙 How to Deploy Your Own ERC-20 Token

In this guide, we'll be setting up an ERC-20 token on the Rinkeby test network - start thinking what name you would like to name your very own ERC-20!

## Guide Requirements

- **[Hardhat](https://hardhat.org/)**: Hardhat is an Ethereum development platform that provides all the tools needed to build, debug and deploy smart contracts.
- **[Alchemy](https://www.alchemy.com/)**: Alchemy is a blockchain development platform from which we will use some APIs to help query the Ethereum blockchain.

## Step 1: Create an Alchemy Account

If you worked through some of the other guides, and you have already set up an account, **you can skip this step**.

1. Go to https://www.alchemy.com/ and create an account (you can use a throwaway email)
2. In the Dashboard, select 'Create App'
3. Enter a name and description, whatever you want to call it, we called it 'ChainShot Interaction Project' and described it as 'Learning how to interact with live smart contracts'
4. Select 'Development' for Environment and 'Kovan' for Network
5. Select 'Create App' and select your newly-created project from the list in your Dashboard

![Alchemy](https://i.imgur.com/R8guVDP.png)

7. Select 'View Key'
8. We will need the 'HTTP' key for the following step, so keep this tab open


## Step 2: Acquire Rinkeby ETH

> **STRONG RECOMMENDATION**: Download a separate browser to the one you regularly use -- presumably, the one that contains a MetaMask wallet you regularly use -- and download the wallet extension there. So if you mainly use Brave Browser, download a Google Chrome browser and install the MetaMask wallet extension there. If you use Safari, download Google Chrome and so on... This can be your browser specific for testing - in this way, we remove any risk of accidentally using funds or private keys associated with your regularly used wallets.

1. As suggested above, hopefully you are in a separate browser to the one you regularly use.
2. In MetaMask, select the icon in the top right and select 'Create Account' - give your wallet a name that you'll remember! We called it "Test Account".
3. Go to https://faucet.rinkeby.io/ - this is an authenticated faucet, meaning you'll have to create a social media post on either Twitter or Facebook that contains the public address where you want to receive the test ether.
    
![Faucet](https://i.imgur.com/0zXRgyi.png)  
    
4. In MetaMask, select the 'Rinkeby Test Network' from the 'Networks' drop-down menu. 
5. Copy-paste your MetaMask public address, from your newly-created test account, into a post on Twitter (this can be done from a throwaway Twitter account) and tweet it! As the faucet website states, the surrounding text doesn't matter; as long as your pubkey is in full format, you are good!
6. Now, copy-paste your tweet post's URL into the Rinkeby faucet input box and select an amount in the 'Give me Ether' dropdown.
7. Your request will be added to the queue and you should see the selected amount transferred to your MetaMask in a few seconds!


## Step 3: Set Up Project Structure Using Hardhat

1. In a folder of your choice, run `mkdir launch-erc20 && cd launch-erc20`
2. Run `npm init -y`
3. Run `npm install --save-dev hardhat`
4. Run `npm install @openzeppelin/contracts @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers`
5. Run `npx hardhat` to initiate the Hardhat development environment - it will bring up some yes/no options, press `Enter` to select them

![Hardhat](https://i.imgur.com/AYePu8i.png)

6. The command above should display the terminal output shown in the image. Use the arrow keys to toggle the options and select `Create an empty hardhat.config.js`
7. Your project directory should now contain the following: `node modules`, `package.json`, `package-lock.json` and the empty `hardhat.config.js` you just created - rolling on!
8. Open your newly created `hardhat.config.js` and copy-paste the following (overwrite the whole file):

```js
require("@nomiclabs/hardhat-waffle");

// TO DO: Copy-paste your Alchemy HTTP Key from Step #1
const ALCHEMY_API_HTTP_KEY = "";

// TO DO: Copy-paste your MetaMask Test Account Rinkeby Private Key **MAKE SURE YOU **
const RINKEBY_PRIVATE_KEY = "";

module.exports = {
  solidity: "0.8.0",
  networks: {
    rinkeby: {
      url: "${ALCHEMY_API_HTTP_KEY}",
      accounts: [`0x${RINKEBY_PRIVATE_KEY}`],
    } 
  }
};

```

9. In the above file, make sure you copy-paste your Alchemy HTTP URL
11. Also, make sure you copy-paste your MetaMask **Rinkeby Test Account** private key into the `RINKEBY_PRIVATE_KEY` variable declaration

> You can get your Rinkeby private key by going to MetaMask > Select the menu icon in top right > Account Details > Export Private Key
10. Once you have done the last two steps, save and close the file
11. Now let's scaffold out more of our project structure: run `mkdir contracts` and `cd` into it
12. Now it's time to name your token! You must match the name of your smart contract with your token name, so if you choose `GoofyGooberToken`, you must name your contract `GoofyGooberToken.sol` - we're all Goofy Goobers!
13. For this guide, we will go with `GoofyGoober` for our token name (same as above, just without the "Token") - thus we will name our contract `GoofyGoober.sol`. Run `touch GoofyGoober.sol` if you would like to follow along or replace the name with yours, be creative!
14. Open the newly-create `.sol` file and copy-paste the following:

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract GoofyGoober is ERC20 {
    uint constant _initial_supply = 100 * (10**18);
    constructor() ERC20("GoofyGoober", "GG") public {
        _mint(msg.sender, _initial_supply);
    }
}
```
> The token symbol you choose, in our case "GG" can be any arbitrary character length but do keep in mind that some UIs may display ones that are too long differently.

15. If you are going with an ERC-20 of your own name (you should!), make sure to change all the Goofy Goober referefences to match the name of your `.sol` file
16. Feel free to edit the initial supply by changing the `100` to how many tokens you would like your initial supply to be - we put 100 because there are very few true Goofy Goobers in the world! You can put any number you'd like for this - make sure to leave the `(10**18)` as that multiplies the number we want as our supply with the `wei` amount, as the Solidity compiler will denominate these numbers in `wei`
17. Save and close the file
18. `cd ..` back into your project root directory and run `mkdir scripts` and `cd` into that directory
19. Run `touch deploy.js`, open the newly-created file and copy-paste the following:

```js
async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const weiAmount = (await deployer.getBalance()).toString();
  
  console.log("Account balance:", (await ethers.utils.formatEther(weiAmount)));
  

  // make sure to replace the "GoofyGoober" reference with your own ERC-20 name!
  const Token = await ethers.getContractFactory("GoofyGoober");
  const token = await Token.deploy();

  console.log("Token address:", token.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
});
```

20. Just one action item: replace the reference to "GoofyGoober" in the file if you went with another name for your ERC-20 token
21. Save and close the file
22. `cd ..` back into your project root directory

## Step 4: Deploy Your ERC-20 Token!

1. Run `npx hardhat run scripts/deploy.js --network rinkeby `
2. Your contract will be compiled and deployed to the Rinkeby network! You should see something similar to this in your terminal output:

![formatEther](https://i.imgur.com/2FXHuVw.png)


3. Go to https://rinkeby.etherscan.io/ and input your outputted Token address to see your deployed ERC-20 contract on Rinkeby!


Now it's time to have the real fun! Send some of your new tokens to your friends and family, stimulate an economy - create the Bitcoin/Ethereum of the future! In this guide, you deployed your own ERC-20 token on Rinkeby using the OpenZeppelin ERC20 standard - great job!

## Step 5: Send Some Tokens!

We are going to challenge you to send some tokens in one of two ways:

1. **More Challenging Way**: Write your own Hardhat Script to do an airdrop!
2. **Simpler Way**: Add your ERC-20 token to MetaMask and send it to an address via the UI!




