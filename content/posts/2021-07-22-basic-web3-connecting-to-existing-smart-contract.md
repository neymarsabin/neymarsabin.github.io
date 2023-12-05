---
layout: post
title: Connect to an Existing Blockchain Using Web3
type: blog
author: neymarsabin
tags: 
- blockchain
- web3
- smart contracts
---

Connecting to an existing blockchain using web3 can sometimes be confusing and difficult to figure out. This is a basic implementation that fetches amount of tokens that have been staked in a DEX. The decentralized exchange is ApeSwapFinance and the method we are going to call is `balanceOf` from this contract. [ApeSwap MasterApe.sol](https://github.com/ApeSwapFinance/apeswap-banana-farm/blob/master/contracts/MasterApe.sol).
Let's start with basic html template. `index.html`
```html
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="./index.css">
    <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
    <title> Web3 Demo ApeFinance BlockChain </title>

  </head>
  <body>
    <div id="amount-details">
      Amount: 0
    </div>
    <script src="./index.js"> </script>
  </body>
</html>
```
We obviously need `web3.min.js` to get all the functions from the library. And we are importing `index.js` which will contain the javascript to connect to the `smart contract`. This is not the best code but it's something that you can start with.
```javascript
window.addEventListener('load', async () => {
  if (window.ethereum) {
		window.web3 = new Web3(window.ethereum);
		window.ethereum.enable();
	} else if (window.web3) {
		window.web3 = new Web3(window.web3.currentProvider);
	} else {
		window.alert(
			"Non-Ethereum browser detected, You Should consider trying Metamask!!"
		);
	};
    
    var contractAddress = '0x603c7f932ED1fc6575303D8Fb018fDCBb0f39a95';
    var abi = JSON.parse(`<Abi that can be copied from the mainnet description >`);
    var contract = new web3.eth.Contract(abi, contractAddress);
    var account;
    web3.eth.getAccounts(function(error, accounts) {
    if(error != null) {
      alert("Error retrieving accounts");
      return;
    };
    if(accounts.length == 0) {
      alert("No account found!!");
    }
    account = accounts[0];
    web3.eth.defaultAccount = account;
    var info = document.getElementById("amount-details");
    contract.methods.balanceOf(account).call().then(function(details) {
      console.log("Amount: ", details);
      info.innerHTML = "Amount: " + details;
    });
  });
});
```
So, basically we need two things to initialize instance of the contract. `contractAddress` and `contractAbi`. We can get the contractAddress and abi's from the explorer like for ApeSwapFinance the link from explorer is. [https://bscscan.com/token/0x603c7f932ed1fc6575303d8fb018fdcbb0f39a95](https://bscscan.com/token/0x603c7f932ed1fc6575303d8fb018fdcbb0f39a95)
And we can find the abi under [https://bscscan.com/address/0x603c7f932ed1fc6575303d8fb018fdcbb0f39a95#code](https://bscscan.com/address/0x603c7f932ed1fc6575303d8fb018fdcbb0f39a95#code). There is an option to copy the Abi of the smart contract. Abi's are in JSON format and contains information on deployments, methods and other stuffs essential to create the contract instance. And after that we just need to call the function written in the smart using the `contract` object we created. 
```javascript
    contract.methods.balanceOf(account).call().then(function(details) {
      console.log("Amount: ", details);
      info.innerHTML = "Amount: " + details;
    });
```
Like this snippets calls the `balanceOf` function sending in the connected address and fetches the details.

