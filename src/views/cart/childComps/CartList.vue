<template>
<!--  <div class="cart-list">-->
    <scroll class="cart-list" ref="scroll">
      <cart-list-item v-for="(item,index) in cartList"
                      :key="index"
                      :product="item"></cart-list-item>
    </scroll>
<!--  </div>-->
</template>

<script>
  import Scroll from "components/common/scroll/Scroll";
  import CartListItem from "./CartListItem";
  import {mapGetters} from 'vuex'
  export default {
    name: "CartList",
    components:{
      Scroll,
      CartListItem
    },
    computed:{
      ...mapGetters(['cartList'])
    },
    activated() {
      this.$refs.scroll.refresh()
    }
  }
</script>

<style scoped>
  .cart-list{
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 89px;
    left: 0;
    right: 0;
    z-index: 1;
  }
  .content{
    height: 100%;
    overflow: hidden;
  }
</style>