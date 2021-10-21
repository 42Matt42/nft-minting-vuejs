<template>
  <main class="mt-16 mx-auto max-w-7xl px-4 sm:mt-24">
    <div class="text-center">
      <h1 class="text-4xl tracking-tight font-extrabold text-gray-900 sm:text-5xl md:text-6xl">
        <span class="block xl:inline gradient-text ">My NFT Collection</span>
      </h1>
      <p class="mt-3 max-w-md mx-auto text-base text-gray-500 sm:text-lg md:mt-5 md:text-xl md:max-w-3xl">
        Each unique. Each beautiful. Discover your NFT today.
      </p>
      <div class="mt-5 max-w-xl mx-auto sm:flex sm:justify-center md:mt-8">
        <div class="rounded-md shadow">
          <button class="cta-button" v-if="currentAccount === ''" @click="connectWallet">
            Connect Wallet</button>
          <button class="cta-button" v-else @click="askContractToMintNft">
            <svg v-if="minting" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <span v-if="minting">Minting NFT</span>
            <span v-else>Mint NFT</span>
          </button>
        </div>
        <div class="mt-3 rounded-md shadow sm:mt-0 sm:ml-3">
          <a href="https://testnets.opensea.io/collection/squarenft-ywrcr5wjne" target="_blank" class="w-full flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-md text-white bg-blue-400 hover:bg-blue-500 md:py-4 md:text-lg md:px-10">
            🌊 View Collection on OpenSea
          </a>
        </div>
      </div>
      <p class="mt-5 max-w-sm mx-auto text-white">
        {{totalNFTsMinted}}/{{mintLimit}} NFTs minted so far
      </p>
    </div>
  </main>
  <div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import myEpicNft from "./utils/MyEpicNFT.json";

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      currentAccount: '',
      CONTRACT_ADDRESS: "0xfeD69c1C57156D9016f29eAA4E169594Fa5AA34D",
      totalNFTsMinted: 0,
      mintLimit: 0,
      minting: false
    }
  },
  methods: {
    async checkIfWalletIsConnected() {
      const { ethereum } = window;

      if (!ethereum) {
        console.log("Make sure you have metamask!");
        return;
      } else {
        console.log("We have the ethereum object", ethereum);
        await this.alertWrongNetwork(ethereum);
      }

      const accounts = await ethereum.request({ method: 'eth_accounts' });

      if (accounts.length !== 0) {
        const account = accounts[0];
        console.log("Found an authorized account:", account);
        this.currentAccount = account;
        await this.setupEventListener();
      } else {
        console.log("No authorized account found")
      }
    },
    async connectWallet() {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }

        await this.alertWrongNetwork(ethereum);

        /*
        * Fancy method to request access to account.
        */
        const accounts = await ethereum.request({ method: "eth_requestAccounts" });

        /*
        * Boom! This should print out public address once we authorize Metamask.
        */
        console.log("Connected", accounts[0]);
        this.currentAccount = accounts[0];
        await this.setupEventListener();
      } catch (error) {
        console.log(error)
      }
    },
    async setupEventListener() {
      // Most of this looks the same as our function askContractToMintNft
      try {
        const { ethereum } = window;

        if (ethereum) {
          // Same stuff again
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS, myEpicNft.abi, signer);

          // THIS IS THE MAGIC SAUCE.
          // This will essentially "capture" our event when our contract throws it.
          // If you're familiar with webhooks, it's very similar to that!
          connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
            this.minting = false
            console.log(from, tokenId.toNumber())
            this.totalNFTsMinted = tokenId.toNumber()
            alert(`Hey there! We've minted your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Here's the link: https://testnets.opensea.io/assets/${this.CONTRACT_ADDRESS}/${tokenId.toNumber()}`)
          });

          console.log("Setup event listener!")

        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error)
      }
    },
    async askContractToMintNft() {
      try {
        const { ethereum } = window;

        if (ethereum) {
          this.minting = true
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS, myEpicNft.abi, signer);

          console.log("Going to pop wallet now to pay gas...")
          const options = {gasLimit: 10000000};
          let nftTxn = await connectedContract.makeAnEpicNFT(options);

          console.log("Mining...please wait.")
          await nftTxn.wait();

          console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);

        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        this.minting = false
        console.log(error)
      }
    },
    async alertWrongNetwork(ethereum) {
      let chainId = await ethereum.request({ method: 'eth_chainId' });
      console.log("Connected to chain " + chainId);

      // String, hex code of the chainId of the Rinkebey test network
      const rinkebyChainId = "0x4";
      if (chainId !== rinkebyChainId) {
        alert("You are not connected to the Rinkeby Test Network!");
      }
    },
    async getMintLimit() {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS,
              JSON.stringify([{
                "inputs": [],
                "name": "getMintLimit",
                "outputs": [
                  {
                    "internalType": "uint256",
                    "name": "",
                    "type": "uint256"
                  }
                ],
                "stateMutability": "pure",
                "type": "function"
              }]), provider);

          const totalMinted = await connectedContract.getMintLimit();

          return totalMinted
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error)
      }
    },
    async getTotalNFTsMintedSoFar() {
      try {
        const { ethereum } = window;

        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS,
              JSON.stringify([{
                "inputs": [],
                "name": "getTotalNFTsMintedSoFar",
                "outputs": [
                  {
                    "internalType": "uint256",
                    "name": "",
                    "type": "uint256"
                  }
                ],
                "stateMutability": "view",
                "type": "function"
              }]), provider);


          const totalMinted = await connectedContract.getTotalNFTsMintedSoFar();

          return totalMinted

        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error)
      }
    }
  },
  async created() {
    await this.checkIfWalletIsConnected()
    this.totalNFTsMinted = await this.getTotalNFTsMintedSoFar()
    this.mintLimit = await this.getMintLimit()
  }
}
</script>

<style>
html {
  background-color: #0d1116;
}

.cta-button {
  @apply w-full flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-md text-white md:py-4 md:text-lg md:px-10;
  background: -webkit-linear-gradient(left, #60c657, #35aee2);
}

#app {
  font-family: -apple-system, Inter, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.gradient-text {
  background: -webkit-linear-gradient(left, #60c657 30%, #35aee2 60%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>
