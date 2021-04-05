<template>
  <div id="app">
    <h1>Shopping Cart Example</h1>
    <hr>
    <h2>Products</h2>
    <ProductList/>
    <hr>
    <ShoppingCart/>
  </div>
</template>

<script>
import ProductList from './ProductList.vue'
import ShoppingCart from './ShoppingCart.vue'
import { useStore } from 'vuex';

export default {
  components: { ProductList, ShoppingCart },
  setup () {
    const store = useStore()

    window.AppStore = store;
  },
  mounted() {
    window.$Store = this.$store;
  },
}
</script>
