<template>
  <view class="video_play">
    <image :src="videoObj.img"></image>

    <!-- 工具栏 开始 -->
    <view class="video_tool">
      <view @click="muted=!muted" :class="['iconfont',muted?'iconjingyin':'iconshengyin']"></view>
      <view class="iconfont iconzhuanfa">
        <!-- open-type="share"微信小程序的api，实现分享 -->
        <button open-type="share"></button>
      </view>
    </view>
    <!-- 工具栏 结束 -->

    <!-- 视频 开始 -->
    <view class="video_wrap">
      <!-- objectFit="fill" 是使得视频撑满 -->
      <video :muted="muted" :src="videoObj.video" objectFit="fill"></video>
    </view>
    <!-- 视频 结束 -->

    <!-- 下载 开始 -->
    <view class="download">
      <view @click="handleDownload" class="download_btn">下载高清</view>
    </view>
    <!-- 下载 结束 -->
  </view>
</template>

<script>
export default {
  data(){
    return {
      videoObj:{},
      muted:false
    }
  },
  onLoad(){
    // 获取全局共享数据
    this.videoObj = getApp().globalData.video;
  },
  methods:{
    // 下载视频
    async handleDownload(){
      await uni.showLoading({title:"下载中"})

      // 1 将远程文件 下载到小程序的内存中
      const { tempFilePath } = (await uni.downloadFile({url: this.videoObj.video}))[1];
      // 2 将内存中的文件下载到本地
      await uni.saveVideoToPhotosAlbum({
        filePath: tempFilePath,
      });

      uni.hideLoading();

      await uni.showToast({title:"下载成功"})
    }
  }
}
</script>

<style lang="scss" scoped>
  .video_play {
    position: relative;
    image {
      position: absolute;
      width: 100vw;
      height: 100vh;
      z-index: -1;
      // css3 滤镜
      filter: blur(20px);
    }

    .video_tool {
      display: flex;
      height: 80rpx;
      justify-content: flex-end;
      .iconfont {
        width: 80rpx;
        color: #fff;
        font-size: 50rpx;
        border-radius: 50%;
        background-color: rgba(0,0,0,.2);
        display: flex;
        justify-content: center;
        align-items: center;
        margin-right: 20rpx;
      }

      .iconzhuanfa{
        position: relative;
        button{
          position: absolute;
          // 高宽同步父元素
          width: 100%;
          height: 100%;
          // opacity 属性设置元素的不透明级别。
          opacity: 0;
        }
      }
    }

    .video_wrap {
      display: flex;
      justify-content: center;
      video {
        width: 360rpx;
        height: 600rpx;
      }
  }

  .download {
    display: flex;
    justify-content: center;
    margin-top: 30rpx;
    .download_btn {
      width: 360rpx;
      height: 80rpx;
      border-radius: 40rpx;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #fff;
      border: 3rpx solid #fff;
      background-color: rgba(0,0,0,.5);
    }
  }
}
</style>
