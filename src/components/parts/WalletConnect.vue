<template>
    <div>
      <h2>Checkout</h2>
      <button v-if="!account" @click="connectWallet">Connect Wallet</button>
      <p v-if="account">Connected: {{ account }}</p>
      <button :disabled="!account" @click="proceedToPayment">
        Proceed to Payment
      </button>
    </div>
  </template>
  
  <script>
  import Web3 from "web3";
  
  export default {
    data() {
      return {
        account: null,
        web3: null,
      };
    },
    methods: {
      async connectWallet() {
        if (window.ethereum) {
          try {
            this.web3 = new Web3(window.ethereum);
            await window.ethereum.request({ method: "eth_requestAccounts" });
            const accounts = await this.web3.eth.getAccounts();
            this.account = accounts[0];
            console.log("Connected account:", this.account);
          } catch (error) {
            console.error("Wallet connection failed:", error);
          }
        } else {
          alert("MetaMask not detected. Please install it.");
        }
      },
      proceedToPayment() {
        if (!this.account) {
          alert("Please connect your wallet before proceeding.");
        } else {
          alert(`Proceeding with wallet: ${this.account}`);
          // Implement blockchain payment logic here
        }
      },
    },
  };
  </script>
  