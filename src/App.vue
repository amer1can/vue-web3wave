<template>
  <img alt="Vue logo" src="./assets/logo.png"><br>
  <button @click="connectToWallet">Connect</button><br>
  <button @click="wave">Make a wave</button>
</template>

<script>

import { ethers } from "ethers";
import abi from "./utils/WavePortal.json"

export default {
  name: 'App',
  data() {
    return {
      contractAddress: "0x39b351095a6e8664a53d38bc0aC231Fb9a21A966",
      contractABI: abi.abi,
    }
  },
  mounted() {
    this.checkIfWalletIsConnected();
  },
  methods: {
    async checkIfWalletIsConnected() {
      try {
        const { ethereum } = window;

        if (!ethereum) {
          console.log("Make sure you have metamask!");
          return;
        } else {
          console.log("We have ethereum object", ethereum);
        }

        const accounts = await ethereum.request({
          method: "eth_accounts"
        });

        if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
          //setCurrentAccount(account)
        } else {
          console.log("No authorized account found");
        }

      } catch (err) {
        console.log(err)
      }
    },
    async connectToWallet() {
      const { ethereum } = window;

      try {
        if (!ethereum) {
          console.log("Get Metamask!");
          return;
        }

        const accounts = await ethereum.request({
          method: "eth_requestAccounts"
        });
        console.log("Connected", accounts[0])
      } catch (err) {
        console.log(err)
      }
    },
    async wave() {
      try {
        const { ethereum } = window;

        if(ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(
              this.contractAddress,
              this.contractABI,
              signer
          );

          let count = await wavePortalContract.getTotalWaves();
          console.log("Retrieved total wave count...", count.toNumber())

          const waveTxn = await wavePortalContract.wave();
          console.log("Mining...", waveTxn.hash);

          await waveTxn.wait();
          console.log("Mined -- ", waveTxn.hash);

          count = await wavePortalContract.getTotalWaves();
          console.log("Retrieved total wave count...", count.toNumber())

        } else {
          console.log("Ethereum object doesn't exist!");
        }

      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
