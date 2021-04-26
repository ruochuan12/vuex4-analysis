<template>
  <div class="cart">
    <h2>Your Cart</h2>
    <p v-show="!products.length"><i>Please add some products to cart.</i></p>
    <ul>
      <li
        v-for="product in products"
        :key="product.id">
        {{ product.title }} - {{ currency(product.price) }} x {{ product.quantity }}
      </li>
    </ul>
    <p>Total: {{ currency(total) }}</p>
    <p><button :disabled="!products.length" @click="checkout(products)">Checkout</button></p>
    <p v-show="checkoutStatus">Checkout {{ checkoutStatus }}.</p>
  </div>
</template>

<script>
import { computed, getCurrentInstance, inject, provide } from 'vue'
import { useStore } from 'vuex'
import { currency } from '../currency'

export default {
  setup () {
    const store = useStore()

    // 若川加入的调试代码--start
    window.ShoppingCartStore = store;
    window.ShoppingCartCurrentInstance = getCurrentInstance();
    provide('weixin', 'ruochuan12');
    provide('weixin1', 'ruochuan12');
    provide('weixin2', 'ruochuan12');
    const mp = inject('ruochuan12');
    console.log(mp, '介绍-ShoppingList'); // 微信搜索「若川视野」关注我，专注前端技术分享。
    // 若川加入的调试代码--start

    const checkoutStatus = computed(() => store.state.cart.checkoutStatus)
    const products = computed(() => store.getters['cart/cartProducts'])
    const total = computed(() => store.getters['cart/cartTotalPrice'])

    const checkout = (products) => store.dispatch('cart/checkout', products)

    return {
      currency,
      checkoutStatus,
      products,
      total,
      checkout
    }
  }
}
</script>
