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
import { getCurrentInstance, provide } from 'vue'
import { useStore } from 'vuex';

export default {
  components: { ProductList, ShoppingCart },
  setup () {
    const store = useStore()
    provide('ruochuan12', '微信搜索「若川视野」关注我，专注前端技术分享。')

    window.AppStore = store;
    window.AppCurrentInstance = getCurrentInstance();
  },
  mounted() {
    window.$Store = this.$store;
  },
}
</script>
