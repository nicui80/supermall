<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" ref="nav" @titleClick="titleClick"></detail-nav-bar>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods-info="goodsInfo"></detail-base-info>
      <detail-shop-info :shop-info="shopInfo"></detail-shop-info>
      <detail-image-info :detail-info="detailInfo" @detailImgLoad="detailImgLoad"></detail-image-info>
      <detail-params-info ref="params" :item-params="itemParams"></detail-params-info>
      <detail-comment-info ref="comment" :comment-info="commentInfo"></detail-comment-info>
      <goods-list ref="recommend" :goods="recommends"></goods-list>
    </scroll>
    <detail-bottom-bar @addToCart="addToCart"></detail-bottom-bar>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
  import DetailNavBar from "./childComps/DetailNavBar";
  import DetailSwiper from "./childComps/DetailSwiper";
  import DetailBaseInfo from "./childComps/DetailBaseInfo";
  import DetailShopInfo from "./childComps/DetailShopInfo";
  import DetailImageInfo from "./childComps/DetailImageInfo";
  import DetailParamsInfo from "./childComps/DetailParamsInfo";
  import DetailCommentInfo from "./childComps/DetailCommentInfo";
  import DetailBottomBar from "./childComps/DetailBottomBar";

  import Scroll from "components/common/scroll/Scroll";
  import GoodsList from "components/content/goods/GoodsList";

  import {getDetail,getRecommend,Goods} from "network/detail";
  import {itemListenerMixin,backTopMixin} from "common/mixin"
  import {debounce} from "common/utils";

  import {mapActions} from 'vuex'
  export default {
    name: "Detail",
    components:{
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailImageInfo,
      DetailParamsInfo,
      DetailCommentInfo,
      DetailBottomBar,
      GoodsList,
      Scroll
    },
    mixins:[itemListenerMixin,backTopMixin],
    data(){
      return{
        iid:null,
        topImages:[],
        goodsInfo:{},
        shopInfo:{},
        detailInfo:{},
        itemParams:{},
        commentInfo:{},
        recommends:[],
        themeTopYs:[],
        getThemeTopY:null,
        currentIndex:0
      }
    },
    created() {
      this.iid = this.$route.params.iid
      this.getDetail(this.iid)
      this.getRecommend()
      this.getThemeTopY = debounce(() => {
        this.themeTopYs=[]
        this.themeTopYs.push(0)
        this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        this.themeTopYs.push(Number.MIN_VALUE)
      },100)
    },
    methods: {
      ...mapActions(['addCart']),
      getDetail(iid){
        getDetail(iid).then(res => {
          const data = res.result
          this.topImages = data.itemInfo.topImages
          this.goodsInfo = new Goods(data.itemInfo, data.columns, data.shopInfo.services)
          this.shopInfo = data.shopInfo
          this.detailInfo = data.detailInfo
          this.itemParams = data.itemParams
          if(data.rate.cRate !== 0){
            this.commentInfo = data.rate.list[0]
          }
          // this.$nextTick(() => {
          //   this.themeTopYs.push(0)
          //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
          //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
          //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
          // })
        })
      },
      getRecommend(){
        getRecommend().then(res => {
          this.recommends = res.data.list
        })
      },
      detailImgLoad(){
        this.refresh()
        this.getThemeTopY()
      },
      titleClick(index){
        this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],500)
      },
      contentScroll(position){
        const positionY = -position.y
        let length = this.themeTopYs.length
        for(let i = 0;i < length-1; i++){
          if(this.currentIndex !== i && (positionY >=this.themeTopYs[i] && positionY < this.themeTopYs[i+1])){
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex;
          }
          // if(this.currentIndex !== i && (i < length-1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1]) || (i === length-1 && positionY >= this.themeTopYs[i])){
          //   this.currentIndex = i
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }
        }
        this.isShowBackTop = (-position.y) > 1000
      },
      addToCart(){
        const product={};
        product.image = this.topImages[0];
        product.title = this.goodsInfo.title;
        product.desc = this.goodsInfo.desc;
        product.price = this.goodsInfo.realPrice;
        product.iid = this.iid;
        // this.$store.commit('addCart',product)
        this.addCart(product).then(res =>{
          this.$toast.show(res,2000)
        })
        // this.$store.dispatch('addCart',product).then(res => {
        //   console.log(res);
        // })
      }
    },
    destroyed() {
      this.$bus.$off('itemImgLoad',this.itemListener)
    }
  }
</script>

<style scoped>
  #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }

  .detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
  .content {
    background-color: #fff;
    height: calc(100% - 93px);
  }
</style>