<template>
  <div class="pay-container" @mouseleave="togglePreview()">
    <!-- TODO: Move to a new component -->
    <ul class="cart-preview" ref="dialog" v-if="cartCount > 0">
      <li v-for="item in cartList" :key="item.name" class="list-item">
        <div><span>{{ item.name }}</span><span class="unit-desc"> x {{ item.quantity }}</span></div>
        <div class="unit-controller">
          <button :aria-label="'Add one ' + item.name" type="button" @click="addOneCartItem(item.name)">+</button>
          <button :aria-label="'Remove one ' + item.name" type="button" @click="removeOneCartItem(item.name)">-</button>
        </div>
      </li>
    </ul>

    <p v-if="!walletConnected" style="color: tomato; text-align: center;">
      Please connect your wallet before proceeding to checkout.
    </p>

    <button 
    data-test="checkout" 
    class="pay" 
    type="button" 
    aria-label="Proceed to checkout" 
    @mouseover="togglePreview()"
    @click="pay()"
    :disabled="!walletConnected">
    Total: {{ currency(total) }}
  </button>

  <!-- <PaymentDetails v-if="walletConnected" :isShow="isShow" @close="closed()" />
  </div>
  <PaymentDetails :isShow="isShow" @close="closed()" /> -->
  
  <PaymentDetails
  v-if="walletConnected"
  :isShow="isShow"
  :orderId="orderId"
  :amountPaid="ethAmount"
  :usdPaid="usDollars"
  :txHash="txHash"
  @close="closed()" />
 </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { mapGetters, mapMutations } from 'vuex';
import { currency, slow } from '../../utils';
import PaymentDetails  from './PaymentDetails.vue';

export default defineComponent({
  name: 'Pay',
  components: { PaymentDetails },
  emits:['store-order'],
  props: {
    isDisablePreview: Boolean,
    walletConnected:{type: Boolean, required: true},
  },
  mounted() {
    console.log(window.ethereum.selectedAddress);
  },
  computed: {
    ...mapGetters({
      total: 'cart/cartTotal',
      cartList: "cart/cartList",
      cartCount: "cart/cartCount"
    })
  },
  data() {
    return {
      isShow: false,
      orderId: '',
      ethAmount: 0,
      usDollars: 0,
      txHash: ''
    };
  },
  methods: {
    ...mapMutations("cart", ["addOneCartItem", "removeOneCartItem"]),
    currency,
    pay() {
      if (!this.walletConnected) {
        alert("Please connect your wallet before proceeding.");
        return;
      }

      const rate = 1655; // rate for 1 eth to dollars when i added this
      this.usDollars = this.total;
      this.ethAmount = parseFloat((this.usDollars / rate).toFixed(4));

      this.orderId = "" + Date.now();

      this.$emit("store-order", this.orderId, this.ethAmount, (txHash: string) => {
        this.txHash = txHash;
        this.ethAmount = this.ethAmount;
        this.isShow = true;
      });

      slow();
      this.isShow = true;
    },
    closed() {
      this.isShow = false;
    },
    togglePreview() {
      if(this.isDisablePreview) return;
      (this.$refs.dialog as HTMLDivElement).classList.toggle('show');
    }
  }
})
</script>

<style scoped>
button.pay {
  border: 4px solid black;
  background: antiquewhite;
  /* margin: 0 6px; */
  font-size: xx-large;
  padding: 6px 10px;
}

button.pay:hover {
  border-color: goldenrod;
  color: goldenrod;
}

button.pay:disabled {
  background: lightgray;
  border-color: gray;
  cursor: not-allowed;
}

.cart-preview.show {
  display: block;
}
.cart-preview {
  display: none;
  padding: 10px;
  background: beige;
  margin-bottom: 10px;
  border: 4px solid black;
  list-style: none;
  min-width: 240px;
}
.pay-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-block: 6px;
}

.unit-controller button {
  min-height: 20px;
  border: 2px solid;
  padding: 0 6px;
  margin: 2px -1px;
  background: antiquewhite;
}
</style>
