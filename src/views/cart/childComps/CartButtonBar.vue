<template>
  <div class="bottom-bar">
    <div class="check-content">
      <check-button :is-checked="isSelectAll"
                    class="check-button"
                    @click.native="checkClick"></check-button>
      <span>全选</span>
    </div>
    <div class="price">
      合计:{{totalPrice}}
    </div>
    <div class="calculate" @click="tipClick">去计算({{checkLength}})</div>
  </div>
</template>

<script>
  import CheckButton from "components/content/checkButton/CheckButton";
  import {mapGetters} from "vuex";
  export default {
    name: "CartButtonBar",
    components:{
      CheckButton
    },
    computed:{
      ...mapGetters(['cartList']),
      totalPrice(){
        return this.$store.state.cartList.filter(item => {
          return item.checked
        }).reduce((preValue, item) =>{
          return preValue + item.price * item.count
        },0)
      },
      checkLength(){
        return this.$store.state.cartList.filter(item => {
          return item.checked
        }).length
      },
      isSelectAll(){
        if(this.cartList.length === 0) return false
        // return !(this.cartList.filter(item => item.checked).length)
        return !this.cartList.find(item => !item.checked)
      }
    },
    methods:{
      checkClick(){
        if(this.isSelectAll){
          this.cartList.forEach(item => item.checked = false)
        }else{
          this.cartList.forEach(item => item.checked = true)
        }
      },
      tipClick(){
        if(!this.isSelectAll){
          this.$toast.show('请选择购买商品',2000)
        }
      }
    }
  }
</script>

<style scoped>
  .bottom-bar{
    position: fixed;
    display: flex;
    width: 100%;
    height: 40px;
    line-height: 40px;
    bottom: 49px;
    background-color: #eeeeee;
    font-size: 14px;
  }
  .check-content{
    display: flex;
    align-items: center;
    margin-left: 10px;
    width: 60px;
  }
  .check-button{
    width: 22px;
    height: 22px;
    line-height: 22px;
    margin-right: 5px;
  }
  .price{
    margin-left:20px;
    flex: 1;
  }
  .calculate{
    width: 90px;
    background-color: red;
    color: #ffffff;
    text-align: center;
  }
</style>