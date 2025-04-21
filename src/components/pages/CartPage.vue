<template>
  <div class="list" v-if="cartList">
    <p v-if="!cartList.length">No coffee, go add some.</p>
    <div v-if="cartList.length">

      <!-- Wallet Connection Button -->
       <button v-if="!account" @click="connectWallet">Connect Wallet</button>
       <p v-if="account">Connected: {{ account }}</p>
      <Pay :isDisablePreview="isHidePayPreview || !account" :walletConnected="account !== null" @store-order="payForOrder"/>

      <ul>
        <li class="list-header">
          <div>Item</div>
          <div>Unit</div>
          <div>Total</div>
          <div></div>
        </li>
        <li v-for="item in cartList" :key="item.name" class="list-item">
          <div>{{ item.name }}</div>
          <div>
            <span class="unit-desc">{{ currency(item.unitPrice) }} x {{ item.quantity }}</span>
            <div class="unit-controller">
              <button :aria-label="'Add one ' + item.name" type="button" @click="addOneCartItem(item.name)">+</button>
              <button :aria-label="'Remove one ' + item.name" type="button" @click="removeOneCartItem(item.name)">-</button>
            </div>
          </div>
          <div>{{ currency(item.price) }}</div>
          <div>
            <button :aria-label="'Remove all ' + item.name" class="delete" type="button" @click="removeCartItem(item.name)">x</button>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { mapActions, mapGetters, mapMutations } from 'vuex';
import { currency } from '../../utils';
import Pay from "../parts/Pay.vue";
import Web3 from "web3";
import { parse } from 'path';

  declare global {
  interface Window {
    ethereum?: any;
  }
}

export default defineComponent({
  name: 'CartPage',
  components: { Pay },
  data() {
    return {
      account: null as string | null,
      web3: null as Web3 | null,
      contract: null as any,
      isHidePayPreview: true,
      contractAddress: "0x1a5951c469d74dff2df5bc34f2c79f322507c2a7",
      abi: [
	{
		"inputs": [],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "string",
				"name": "label",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "string",
				"name": "orderId",
				"type": "string"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "value",
				"type": "uint256"
			}
		],
		"name": "Debug",
		"type": "event"
	},
	{
		"anonymous": false,
		"inputs": [
			{
				"indexed": false,
				"internalType": "string",
				"name": "orderId",
				"type": "string"
			},
			{
				"indexed": true,
				"internalType": "address",
				"name": "customer",
				"type": "address"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "totalAmount",
				"type": "uint256"
			},
			{
				"indexed": false,
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"name": "OrderStored",
		"type": "event"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "_orderId",
				"type": "string"
			}
		],
		"name": "storeOrder",
		"outputs": [],
		"stateMutability": "payable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "withdraw",
		"outputs": [],
		"stateMutability": "nonpayable",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "getContractBalance",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "_orderId",
				"type": "string"
			}
		],
		"name": "getOrder",
		"outputs": [
			{
				"components": [
					{
						"internalType": "string",
						"name": "orderId",
						"type": "string"
					},
					{
						"internalType": "address",
						"name": "customer",
						"type": "address"
					},
					{
						"internalType": "uint256",
						"name": "totalAmount",
						"type": "uint256"
					},
					{
						"internalType": "uint256",
						"name": "timestamp",
						"type": "uint256"
					}
				],
				"internalType": "struct OrderStorage.Order",
				"name": "",
				"type": "tuple"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "string",
				"name": "",
				"type": "string"
			}
		],
		"name": "orders",
		"outputs": [
			{
				"internalType": "string",
				"name": "orderId",
				"type": "string"
			},
			{
				"internalType": "address",
				"name": "customer",
				"type": "address"
			},
			{
				"internalType": "uint256",
				"name": "totalAmount",
				"type": "uint256"
			},
			{
				"internalType": "uint256",
				"name": "timestamp",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "owner",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	}
]
  }
},
  computed: {
    // Option 2
    ...mapGetters({
      cartList: "cart/cartList"
    }),
  },
  // data() {
  //   return {
  //     cartList: null
  //   }
  // },
  methods: {
    currency,
    ...mapActions("cart", []),
    ...mapMutations("cart", ["addOneCartItem", "removeOneCartItem", "removeCartItem"]),

    async connectWallet() {
  if (typeof window.ethereum !== "undefined") {
    try {
      this.web3 = new Web3(window.ethereum);
      await window.ethereum.request({ method: "eth_requestAccounts" });
      const accounts = await this.web3.eth.getAccounts();
      this.account = accounts[0];

      this.contract = new this.web3.eth.Contract(this.abi, this.contractAddress);
    } catch (error) {
      console.error("Wallet connection failed:", error);
    }
  } else {
    alert("MetaMask not detected.");
  }
},

async payForOrder(orderId: string, ethAmount: number, callback: (txHash: string, ethConfirmed: number) => void) {
  if (!this.contract || !this.web3 || !this.account) {
    alert("Web3 or contract not initialized");
    return;
  }

  const roundedETH = parseFloat(ethAmount.toFixed(6));
  const valueInWei = this.web3.utils.toWei(roundedETH.toString(), "ether");

  try {
    const result = await this.contract.methods
      .storeOrder(orderId)
      .send({ from: this.account, value: valueInWei });

    console.log("Payment Success:", result);
    const ethConfirmed = this.web3.utils.fromWei(valueInWei, "ether");
    callback(result.transactionHash, parseFloat(ethConfirmed));
  } catch (error: any) {
    console.error("Payment failed:", error);
    alert("Payment failed: " + error.message);
  }
}

//   async storeOrder(orderId: string, totalAmount: number) {
//   console.log("storeOrder() called with:", orderId, totalAmount);

//   if (!this.web3) {
//     alert("Web3 not initialized.");
//     return;
//   }

//   if (!this.contract || !this.account) {
//     alert("Contract is not initialised");
//     return;
//   }

//   try {
//     const ethAmount = totalAmount * 0.01; //converting to eth so the total isnt alot
//     const totalInWei = this.web3.utils.toWei(ethAmount.toString(), "ether");

//     console.log("Account:", this.account);
//     console.log("Contract:", this.contract.options.address);
//     console.log("Order ID:", orderId);
//     console.log("Amount in ETH:", ethAmount);
//     console.log("Amount in Wei:", totalInWei);

//     const response = await this.contract.methods
//       .storeOrder(orderId)
//       .send({
//         from: this.account,
//         value: totalInWei
//       });

//     console.log("✅ Transaction success:", response);
//     alert("✅ Order stored on blockchain!");
//   } catch (error: any) {
//     console.error("❌ Failed to store order:", error);
//     if (error?.message) {
//       alert("Transaction failed: " + error.message);
//     } else {
//       alert("❌ Transaction failed. See console for details.");
//     }
//   }
// }
},
  created() {
    // setTimeout(() => {
    //   this.cartList = this.$store.getters['cart/cartList'];
    // }, 3000) as any;
  }
})
</script>

<style scoped>
p {
  text-align: center;
  font-size: xx-large;
}

.list {
  margin: 0 auto;
  max-width: 680px;
  padding: 10px;
}

ul {
  padding: 0;
}

li {
  padding: 10px;
  font-weight: 300;
  font-size: x-large;
  display: grid;
  grid-template-columns: repeat(3, auto);
  grid-template-areas: "a a d" "b b c";
}

li {
  border-bottom: 4px dashed lightgray;
}

/* title */
li div:first-child {
  grid-area: a;
}

/* unit price */
li div:nth-child(2) {
  grid-area: b;
  display: flex;
}

/* total */
li div:nth-child(3) {
  grid-area: c;
  justify-self: end;
}

/* delete button */
li div:last-child {
  grid-area: d;
  justify-self: right;
  display: flex;
  align-items: center;
}

.list-header {
  display: none;
}

@media (min-width: 500px) {
  li {
    grid-template-columns: repeat(3, auto);
    /* autoprefixer: off */
    grid-template-areas: "a a d" "b b c";
  }
}

@media (min-width: 800px) {
  li {
    grid-template-columns: 260px 200px 140px 50px;
    /* autoprefixer: off */
    grid-template-areas: "a b c d";
  }

  .list-header {
    display: grid;
    font-weight: bold;
    font-size: initial;
  }
}

/* delete button */

.delete {
  height: 24px;
  width: 24px;
  font-size: 14px;
  border-radius: 50%;
  font-weight: bold;
  background: rgba(10, 10, 10, 0.15);
  color: white;
  font-weight: bold;
  border: none;
}

.delete:hover {
  color: red;
  background: lightpink;
}

/* unit controller */

.unit-controller {
  margin-left: 10px;
}

.unit-controller button {
  min-height: 20px;
  font-size: 14px;
  border: 2px solid;
  padding: 0 6px;
  margin: 2px -1px;
  background: antiquewhite;
}

.unit-desc {
  min-width: 120px;
}

:deep(.pay-container) {
  padding: 0;
}

:deep(.pay) {
  align-self: flex-start;
}
</style>
