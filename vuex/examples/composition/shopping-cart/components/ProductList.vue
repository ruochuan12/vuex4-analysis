<template>
  <ul>
    <li
      v-for="product in products"
      :key="product.id">
      {{ product.title }} - {{ currency(product.price) }}
      <br>
      <button
        :disabled="!product.inventory"
        @click="addProductToCart(product)">
        Add to cart
      </button>
    </li>
  </ul>
</template>

<script>
import { computed, getCurrentInstance, inject, provide } from 'vue'
import { useStore } from 'vuex'
import { currency } from '../currency'

export default {
  setup () {
    const store = useStore()

    // 若川加入的调试代码--start
    window.ProductListStore = store;
    window.ProductListCurrentInstance = getCurrentInstance();
    provide('weixin-2', 'ruochuan12');
    provide('weixin-3', 'ruochuan12');
    provide('weixin-4', 'ruochuan12');
    const mp = inject('ruochuan12');
    console.log(mp, '介绍-ProductList'); // 微信搜索「若川视野」关注我，专注前端技术分享。
    // 若川加入的调试代码---end

    const products = computed(() => store.state.products.all)

    const addProductToCart = (product) => store.dispatch('cart/addProductToCart', product)

    store.dispatch('products/getAllProducts')

    return {
      products,
      addProductToCart,
      currency
    }
  }
  // computed: mapState({
  //   products: state => state.products.all
  // }),
  // methods: mapActions('cart', [
  //   'addProductToCart'
  // ]),
  // created () {
  //   this.$store.dispatch('products/getAllProducts')
  // }
}
</script>
