📒 Read from the blockchain through our website
-----------------------------------------------

Awesome. We made it. We've deployed our website. We've deployed our contract. We've connected our wallet. Now we need to actually call our contract from our website using the credentials we have access to now from Metamask!

So, our smart contract has this function that retrieves the total number of waves.

```solidity
  function getTotalWaves() view public returns (uint256) {
      console.log("We have %d total waves!", totalWaves);
      return totalWaves;
  }
```

Lets call this function from our website :).

Go ahead and write this function right under our `connectWallet()` function.

```jsx
const wave = async () => {
    try {
      const { ethereum } = window;

      if (ethereum) {
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const waveportalContract = new ethers.Contract(contractAddress, contractABI, signer);

        let count = await waveportalContract.getTotalWaves();
        console.log("Retrieved total wave count...", count.toNumber());
      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
}
```

Connect this function to our wave button.

```html
<button className="waveButton" onClick={wave}>
    Wave at Me
</button>
```

Awesome.

So, right now this code **breaks**. In our replit shell it'll say:

![](https://i.imgur.com/JP2rryE.png)

We need those two variables!!

So, contract address you have -- right? Remember when you deployed your contract and I told you to save the address? This is what it's asking for!

But, what's an ABI? Much earlier I mentioned how when you compile a contract, it creates a bunch of files for you under `artifacts`. An ABI is one of those files.

🏠 Setting Your Contract Address
-----------------------------
 
Remember when you deployed your contract to the Rinkeby Testnet (epic btw)? The output from that deployment included your smart contract address which should look something like this:

```
Deploying contracts with the account: 0xF79A3bb8d5b93686c4068E2A97eAeC5fE4843E7D
Account balance: 3198297774605223721
WavePortal address: 0xd5f08a0ae197482FA808cE84E00E97d940dBD26E
```

You need to get access to this in your React app. It's as easy as creating a new property in your `App.js` file called `contractAddress` and setting it's value to the `WavePortal address` thats printed out in your console:

```javascript
import React, { useEffect, useState } from "react";
import { ethers } from "ethers";
import './App.css';

const App = () => {
  const [currentAccount, setCurrentAccount] = useState("");
  /**
   * Create a varaible here that holds the contract address after you deploy!
   */
  const contractAddress = "0xd5f08a0ae197482FA808cE84E00E97d940dBD26E";
```

🛠 Getting ABI File Content
---------------------------
**Rather watch me go through this? [Click here](https://www.loom.com/share/ddecf3caf54848a3a01edd740683ec48)!**

Look at you, already half way down here! Let's move back to our Smart Contract.

When you compile your smart contract, the compiler spits out a bunch of files needed so the Blockchain can actually read the code you wrote. You can find these files in the `artificats` folder located in the root of your Solodity project.

Now, you are looking for the ABI content right? This is going to be given to you in a fancy JSON file under:

`artifacts/contracts/WavePortal.sol/WavePortal.json`


Nice! So what is the point of this thing? Essentially, this file tells your frontend what it can do with your Smart Contract. So, the question becomes how do we get this JSON file into our frontend? For this project we are going to do some good old "copy pasta"!

Copy the contents from your `WavePortal.sol` in your Solidity project and then head to your React App. You are going tro make a new file called `WavePortal.json` in the following path:

`src/utils/WavePortal.json`


You know the drill, paste the ABI file contents right there!

Now that you have your file with all your ABI content ready to go, it's time to import it into your `App.js` file. Right under where you imported `App.css` go ahead and import your JSON file like so:

`import waveportal from './utils/WavePortal.json';`

Now that you have this imported and ready to go, you need actually access the ABI content in your code! If you noticed, the contents of that JSON file you imported as a key called `abi`. Thats exactly what you will be accessing in your code here! Let's take a look at where you are actually using this ABI content:

```javascript
const wave = async () => {
    try {
      const { ethereum } = window;

      if (ethereum) {
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();

        /*
        * You are defining contractABI right here. Let's change this!
        */
        const waveportalContract = new ethers.Contract(contractAddress, contractABI, signer);

        let count = await waveportalContract.getTotalWaves();
        console.log("Retrieved total wave count...", count.toNumber());

        const waveTxn = await waveportalContract.wave();
        console.log("Mining...", waveTxn.hash);

        await waveTxn.wait();
        console.log("Mined -- ", waveTxn.hash);

        count = await waveportalContract.getTotalWaves();
        console.log("Retrieved total wave count...", count.toNumber());
      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
  }
  ```

  Now that you have a way to access your ABI file let's swap out the property `contractABI` with `waveportal.abi` to have it look something like this:
  
  `const waveportalContract = new ethers.Contract(contractAddress, waveportal.abi, signer);`

  Wow. Feels good to not see errors right?


Once you add that file and click the "Wave" button -- **you'll be officially reading data from your contract on the blockchain through your web client**. 

📝 Writing data
---------------

The code for writing data to our contract isn't super different from reading data. The main difference is that when we want to write new data to our contract, we need to notify the miners so that the transaction can be mined. When we read data, we don't need to do this. Reads are "free" because all we're doing is reading from the blockchain, **we're not changing it. **

Here's the code to wave:

```javascript
const wave = async () => {
    try {
      const { ethereum } = window;

      if (ethereum) {
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const waveportalContract = new ethers.Contract(contractAddress, contractABI, signer);

        let count = await waveportalContract.getTotalWaves();
        console.log("Retrieved total wave count...", count.toNumber());

        /*
        * Execute the actual wave from your smart contract
        */
        const waveTxn = await waveportalContract.wave();
        console.log("Mining...", waveTxn.hash);

        await waveTxn.wait();
        console.log("Mined -- ", waveTxn.hash);

        count = await waveportalContract.getTotalWaves();
        console.log("Retrieved total wave count...", count.toNumber());
      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
  }
```

Pretty simple, right :)?

What's awesome here is while the transaction is being mined you can actually print out the transaction hash, copy/paste it to [Etherscan](https://rinkeby.etherscan.io/), and see it being processed in real-time :).

When we run this, you'll see that total wave count is increased by 1. You'll also see that Metamask pops us and asks us to pay "gas" which we pay for using our fake $. I'll talk about this on the live stream a lot more. There is a great article on it [here](https://ethereum.org/en/developers/docs/gas/).

🎉 Success
----------

**NICEEEEEEE :).**

Really good stuff. We now have an actual client that can read and write data to the blockchain. From here, you can do whatever you want. You have the basics down. You can build a decentralized version of Twitter. You can build a way for people to post their favorite memes and allow people to "tip" the people who post the best memes with Ethereum. You can build a decentralized voting system that a country can use to vote in a politician where everything is open and clear. 

The possibilities are truly endless.

🚨 Required: Before you click "Next Lesson"
-------------------------------------------

*Note:if you don't do this, Farza will be very sad :(.*

Customize your site a little to show the total number of waves. Maybe show a loading bar while the wave is being mined, whatever you want. Do something a little different!

Once you feel like you're ready, share the link to your website with us in #course-chat so we can connect our wallets and wave at you :).

🎁 Wrap Up
--------------------

You are on your way to conquering the decentralized web. IMPRESSIVE. Take a look at all the code you wrote in this section by visiting [this link](https://gist.github.com/adilanchian/71890bf4fcd8f78e94c77cf694b24659) to make sure you are on track with your code!