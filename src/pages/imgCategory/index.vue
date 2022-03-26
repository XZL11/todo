<template>
  <view>
    <view class="category_tab">
      <view class="category_tab_title">
        <view class="title_inner">
          <uni-segmented-control
            :values="items.map(v=>v.title)"
            :current="current"
            @clickItem="onclickItem"
            style-type="text"
            active-color="#d4237a"
          ></uni-segmented-control>
        </view>
        <view class="iconfont iconsearch"></view>
      </view>
      <scroll-view @scrolltolower="handleToLower" enable-flex scroll-y class="category_tab_content">
        <view class="cate_item"
          v-for="item in vertical"
          :key="item.id"
          @touchstart="handleTouchstart"
          @touchend="handleTouchend"
        >
          <image mode="widthFix" :src="item.thumb"></image>
        </view>
      </scroll-view>
    </view>
  </view> 
</template>

<script>
import {uniSegmentedControl} from '@dcloudio/uni-ui'
export default {
  components: {
    uniSegmentedControl,
  },
  data() {
    return {
      items: [
        {title:"最新",order:"new"},
        {title:"热门",order:"hot"}
      ],
      current: 0,
      params:{
        limit:30,
        skip:0,
        order:"new"
      },
      id:0,
      vertical:[],
      hasMore:true,
      startTime:0
    };
  },
  onLoad(options){
    this.id=options.id;
    this.getList()
  },
  methods: {
    onclickItem(e) {
     
      if (this.current !== e.currentIndex) {
        this.current = e.currentIndex;
      }
       /*
        1.根据被点击的标题 切换标题
        2.切换 order
        3.重新发送请求
        */
      this.params.order=this.items[e.currentIndex].order;
      this.vertical=[]
      this.getList()
    },
    getList(){
      this.request({
        url:`https://service.picasso.adesk.com/v1/vertical/category/${this.id}/vertical`,
        data:this.params
      })
      .then(result=>{
        console.log(result);
        // 判断还有没有下一页
        if(result.res.vertical.length === 0){
          this.hasMore = false;
          //  弹窗提示用户
          uni.showToast({
            title:"没有数据了！",
            icon:"none"
          })
          return;
        }
        // 数据请求
        if (this.vertical.length === 0) {
          this.vertical = result.res.vertical;
        }
        // 进行数组拼接
        this.vertical = [...this.vertical,...result.res.vertical]
      })
    },
    // 滚动条触底事件
    handleToLower() {
      /*
        1.修改参数 skip+=limit
        2.重新发送请求 getList()
        3.请求回来成功 hots 数据得叠加
      */
     if(this.hasMore){
      this.params.skip += this.params.limit;
      this.getList();
     }else{
       return 
     }
      
    },

    // 按下
    handleTouchstart(event){
      // console.log("点击屏幕");
      this.startTime = Date.now();

    },
    handleTouchend(event){
      // console.log("离开屏幕");
      const endTime = Date.now()
      // 判断按下的时长
      console.log(endTime - this.startTime);
      if(endTime - this.startTime > 1000){
        this.handleDownload()
      }  
    },

     

    // 点击下载图片 async await promise
    async handleDownload() {
      // uni.downloadFile
      // uni.saveImageToPhotosAlbum

      await uni.showLoading({
        title:'下载中'
      })

      // 1.将远程文件下载到小程序的内存中 tempFilePath
      const result1 = (await uni.downloadFile({url:this.vertical.thumb}));
      const { tempFilePath } =result1;

      // 2.将小程序内存中的临时文件下载到本地上
      const result2 = await uni.saveImageToPhotosAlbum({ filePath:tempFilePath })

      // 3.提示用户下载成功
      uni.hideLoading();
      await uni.showToast({
        title:'下载成功'
      })
    }
  },
}
</script>


<style lang="scss">
  .category_tab {
    .category_tab_title {
      position: relative;
      .title_inner{
        width: 60%;
        margin: 0 auto;
      };
      .iconsearch{
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        right: 5%;
      };
    };
    .category_tab_content{
      display: flex;
      flex-wrap: wrap;
      height: calc( 100vh - 36px );
     .cate_item {
        width: 33.33%;
        border: 5rpx solid #fff;
        image {
          
        }
      }
    }
  };
</style>
