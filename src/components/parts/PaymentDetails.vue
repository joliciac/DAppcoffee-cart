<template>
  <div class="modal" v-show="isShow">
    <div class="modal-content size">
      <!-- <span class="close" @click="closeModal()">&times;</span> -->
      <section>
        <h1>Payment Confirmation</h1>
        <button class="close" @click="closeModal()">&times;</button>
      </section>

      <form aria-label="Payment form">
        <p>Great coffee selection! Here is your order summary:</p>
        <p><strong>Order ID:</strong> <code>{{ orderId }}</code></p>
        <p><strong>Amount Paid:</strong> {{ currency(amountPaid) }} ETH <span style="color:gray;">(~${{ usdPaid?.toFixed(2) }})</span></p>
        <p v-if="txHash"><strong>Txn Hash:</strong> {{ txHash }}</p>
        
        <div style="text-align: right;">
          <button @click="closeModal()">Done</button>
        </div>
</form>

    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { currency } from '../../utils.js';

export default defineComponent({
  name: 'PaymentDetails',
  props: {
    isShow: Boolean,
    orderId: String,
    amountPaid: Number,
    usdPaid: Number,
    txHash: String},
  emits: ['close'],
  methods: {
    currency,
    closeModal() {
      this.$emit('close')
    }
  }
});
</script>

<style scoped>
.modal {
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  padding-top: 100px; /* Location of the box */
  left: 0;
  top: 0;
  width: 100%;  /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0,0,0); /* Fallback color */
  background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
}

/* Modal Content */
.modal-content {
  background-color: #fefefe;
  margin: auto;
  padding-inline: 20px;
  padding-block: 10px;
  /* border: 1px solid #888; */
  width: 80%;
  max-width: 600px;
  min-width: 300px;
  border: 4px solid black;
}

/* The Close Button */
.close {
  color: slategrey;
  float: right;
  font-size: 28px;
  font-weight: bold;
  border: none;
  background: transparent;
}

.close:hover,
.close:focus {
  color: #000;
  text-decoration: none;
  cursor: pointer;
}

h1 {
  margin-block: 16px;
}

section {
  display: flex;
  justify-content: space-between;
}

form label {
  margin-inline-end: 10px;
  padding-block: 10px;
}

form input[type="checkbox"] {
  margin-inline-end: 10px;
}

form input[type="email"], form input[type="text"] {
  padding-block: 4px;
  padding-inline: 10px;
}

form div {
  padding-block: 10px;
}

form div:last-of-type {
  display: flex;
  justify-content: flex-end;
}

form button {
  border: 4px solid black;
  background: antiquewhite;
  margin: 0 6px;
  font-size: x-large;
}

form button:hover {
  border-color: goldenrod;
  color: goldenrod;
}

p {
  margin-block: 10px;
}

.size {
  animation: mymove 2s infinite;
}

@keyframes mymove {
  from {background-color: rgb(127, 195, 179); }
  to {background-color: rgb(222, 98, 38);}
  /* to {background-color: rgb(178, 187, 140);} */
}
</style>
