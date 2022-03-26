<template>
  <scroll-view
    scroll-y
    enable-flex
    class="video_main"  
    @scrolltolower='handleScrolltolower'
  >
    <view 
      class="video_item"
      v-for="item in videowp"
      :key="item.id"
      @click="handleGoVideo(item)"
    >
      <image mode="widthFix" :src="item.img"></image>
    </view>
  </scroll-view>
</template>

<script>
export default {
  // 接收父组件传回来的urlobj
  props:{
    urlobj:Object
  },
  data() {
    return {
      videowp:[],
      hasMore:true
    }
  },
  watch:{
    urlobj(){
      // 数据发生变化之后
      // 清空数组
      this.videowp=[],
      this.hasMore=true,
      this.getList()
    }
  },
  mounted(){
    this.getList()
  },
  methods:{
    // 请求数据
    getList(){
      this.request({
        url:this.urlobj.url,
        data:this.urlobj.params
      })
      .then(result=>{
        // console.log(result);
        if (result.res.videowp.length === 0){
          this.hasMore = false;
          uni.showToast({
            title:'没有更多数据了！',
            icon:'none'
          })
          return;
        }
        this.videowp = result.res.videowp
      })
    },
    // 分页时间
    handleScrolltolower(){
      if (this.hasMore){
        this.urlobj.params.skip+=this.urlobj.params.limit;
        this.getList()
      }else{
        uni.showToast({
          title:'没有了',
          icon:'none'
        })
      }
    },
    // 图片点击事件
    handleGoVideo(item){
      // 1 将数据储存到全局中
      getApp().globalData.video = item;
      // 2 页面跳转
      uni.navigateTo({
        url:'/pages/videoPlay/index'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .video_main {
    display: flex;
    flex-wrap: wrap;  //换行
    height: calc( 100vh - 36px );
    .video_item {
      width: 33.33%;
      border: 5rpx solid #fff;
      image {

      }
  }
}
</style>
