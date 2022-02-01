<template>
  <img alt="Vue logo" src="./assets/logo.png"><br>
  <button @click="connectToWallet" class="btn btn-info">{{ connectStatus }}</button><br>
  <button @click="wave" class="btn btn-dark mt-2">Make a wave</button>

  <div class="waves" v-if="allWaves.length" >
    <div class="mt-5 border border-dark d-flex flex-column p-4" v-for="wave in allWaves" :key="wave">
      <div class="fw-bold">{{ wave.address }}</div>
      <div class="fst-italic">{{ wave.timestamp }}</div>
      <div class="fs-4">{{ wave.message }}</div>
    </div>
  </div>
  <div v-else class="waves">
    <h2 class="mt-5">No waves yet...</h2>
  </div>
</template>

<script>

import { ethers } from "ethers";
import abi from "./utils/WavePortal.json"

export default {
  name: 'App',
  data() {
    return {
      contractAddress: "0x3ce690960d2684eD7BB760C1E14a9b0a5Af14867",
      contractABI: abi.abi,
      connectStatus: 'Connect',
      allWaves: []
    }
  },
  mounted() {
    this.checkIfWalletIsConnected();
  },
  methods: {
    async getAllWaves() {
      this.allWaves = [];
      try {
        const { ethereum } = window;
        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const wavePortalContract = new ethers.Contract(this.contractAddress, this.contractABI, signer);

          const waves = await wavePortalContract.getAllWaves();

          waves.forEach(wave => {
            this.allWaves.push({
              address: wave.waver,
              timestamp: new Date(wave.timestamp * 1000),
              message: wave.message
            })
          })
        }

      } catch (err) {
        console.log(err)
      }

    },
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
          this.connectStatus = "Connected";
          await this.getAllWaves();
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

          const waveTxn = await wavePortalContract.wave("Message here!");
          console.log("Mining...", waveTxn.hash);

          await waveTxn.wait();
          console.log("Mined -- ", waveTxn.hash);

          count = await wavePortalContract.getTotalWaves();
          console.log("Retrieved total wave count...", count.toNumber())

          await this.getAllWaves();

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
.waves {
  max-width: 600px;
  margin: 0px auto;
}
</style>
