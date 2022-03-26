<template>
  <view>
    <!-- 专辑背景 开始 -->
      <view class="album_bg">
        <image mode="widthFix" :src="album.cover"></image>
        <view class="album_info">
          <view class="album_name">{{album.name}}</view>
          <view class="album_btn">关注专辑</view>
        </view>
      </view>
    <!-- 专辑背景 结束 -->

    <!-- 专辑作者 开始 -->
    <view class="album_author">
      <view class="album_author_info">
        <image mode="widthFix" :src="album.user.avatar"></image>
        <view class="album_author_name">{{album.user.name}}</view>
      </view>
        <view class="album_desc">
          <!-- text 可以识别文本自带的分行符号 -->
          <text>{{album.desc}}</text>
        </view>
    </view>
    <!-- 专辑作者 结束 -->

    <!-- 列表 开始 -->
    <view class="album_list">
      <view class="album_item"
        v-for="item in wallpaper"
        :key="item.id"
      >
        <go-detail :list="wallpaper" :index="index">
          <image mode="widthFix" :src="item.thumb+item.rule.replace('$<Height>',360)"></image>
        </go-detail>
      </view>
    </view>
    <!-- 列表 结束 -->
  </view>
</template>

<script>
import goDetail from "@/components/goDetail"
export default {
  components: {
    goDetail
  },
  data() {
    return {
      params: {
        limit: 30,
        order:"new",
        skip:0,
        // 1 返回值中有 album 对象 表示第一次请求数据
        // 0 返回值中 只有 wallpaper 数组 不是第一次请求数据
        first:1
      },
      id:0,
      album:{},
      wallpaper:[],
      hasMore:true
      
    }
  },
  onLoad(options){
    this.id = options.id
    // this.id = "4cf727fa716ec22db1000000"
    this.getList()
  },
  // 页面触底 上拉加载更多
  onReachBottom(){
    if(this.hasMore){
      this.params.first = 0
      this.params.skip += this.params.limit
      this.getList()
    }else{
      
    }
  },
  methods: {
    getList(){
      this.request({
        url: `http://service.picasso.adesk.com/v1/wallpaper/album/${this.id}/wallpaper`,
        data: this.params
      })
      .then(result=>{
        console.log(result);
        if(Object.keys(this.album).length===0){
          this.album = result.res.album;
        }
        if(result.res.wallpaper.length===0){
          this.hasMore = false;
          uni.showToast({
            title: '没有数据了',
            icon: 'none'
          });
          return
        }
        
        this.wallpaper = [...this.wallpaper, ...result.res.wallpaper]
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .album_bg {
    position: relative;
    image {

    }

    .album_info {
      position: absolute;
      width: 100%;
      left: 0;
      bottom: 0;
      display: flex;
      justify-content: space-between;
      height: 80rpx;
      align-items: center;
      color: #fff;
      padding: 0 24rpx;
      .album_name {
        font-size: 40rpx;
      }

      .album_btn {
        background-color: $color;
        width: 152rpx;
        height: 60rpx;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 10rpx;
      }
    }
  }

  .album_author {
    .album_author_info {
      display: flex;
      padding: 10rpx;
      align-items: center;
      image {
        width: 60rpx;
      }

      .album_author_name {
        font-size: 35rpx;
        color: #000;
      }
    }

    .album_desc {
      font-size: 30rpx;
      padding: 0 10rpx;
    }
  }

  .album_list {
    display: flex;
    flex-wrap: wrap;
    .album_item {
      width: 33.33%;
      border: 3rpx solid #fff;
      image {

      }
    }
  }
</style>
