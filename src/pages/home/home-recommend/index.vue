<template>
  <scroll-view @scrolltolower="handleToLower" class="recommend_view" scroll-y v-if="recommend.length>0">
  <view v-if="recommend.length > 0">
    <!-- 推荐 开始 -->
    <view class="recommend_wrap">
      <navigator 
        class="recommend_item"
        v-for="item in recommend"
        :key="item.id"
        :url="`/pages/album/index?id={item.target}`"
      >
        <!-- mode="widthFix" 是长宽高自适应，不被压缩 -->
        <image mode="widthFix" :src="item.thumb"></image>
      </navigator>
    </view>
    <!-- 推荐 结束 -->

    <!-- 月份 开始 -->
    <view class="moneths_wrap">
      <view class="moneths_title">
        <view class="moneths_title_info">
          <view class="moneths_info">
            <text> {{moneths.MM}}/ </text>
            {{moneths.DD}} 月
          </view>
          <view class="moneths_text">{{moneths.title}}</view>
        </view>
        <view class="moneths_title_more">更多>></view>
      </view>
      <view class="moneths_content">
        <view class="moneths_item"
        v-for="(item,index) in moneths.items"
        :key="item.id"
        >
        <!-- index 是索引 -->
        <go-detail :list="monthes.item" :index="index">
          <image mode="widthFix" :src="item.thumb+item.rule.replace('$<Height>',360)"></image>
        </go-detail>
        </view>
      </view>
    </view>
    <!-- 月份 结束 -->

    <!-- 热门 开始 -->
     <view class="hots_wrap">
       <view class="hots_title">
         <text> 热门 </text>
       </view>
       <view class="hots_content">
         <view 
          class="hots_item"
          v-for="item in hots"
          :key="item.id"
         >
          <go-detail :list="hots" :index="index">
            <image
              mode="widthFix"
              :src="item.thumb"
            ></image>
          </go-detail>
         </view>
       </view>
     </view>
    <!-- 热门 结束 -->
  </view>
  </scroll-view>
</template>

<script>
import moment from "moment"
import goDetail from "@/components/goDetail"

export default {
  components:{
    goDetail
  },
  data() {
    return {
      // 推荐列表
      recommend:[],
      // 月份
      moneths:{},
      // 热门
      hots:[],
      // 请求的参数
      params:{
        // 要获取多少条
        limit: 30,
        // 关键字
        order: "hot",
        // 要跳过几条
        skip: 0
      },
      hasMore:true
    }
  },
  mounted() {
    // 调用接口
    this.getList();
  },
  methods: {
    // 获取接口数据
    getList() {
      this.request({
        url:"http://service.picasso.adesk.com/v3/homepage/vertical?limit=30&order=hot&skip=0",
        data: this.params,
      })
      .then(result=>{
        // console.log(result)

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

        if (this.recommend.length === 0) {
          // 第一次发送请求
          // 推荐模块
          this.recommend = result.res.homepage[1].items;
          // 月份模块
          this.moneths = result.res.homepage[2]
         
          // 时间戳
          this.moneths.MM = moment(this.moneths.stime).format("MM")
          this.moneths.DD = moment(this.moneths.stime).format("DD")
        }
          // 热门模块
          // 进行数组拼接
          this.hots = [...this.hots,...result.res.vertical]
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

     }
      
    }
  }
}
</script>

<style lang="scss" scoped>
  .recommend_view {
    // height: 屏幕得高度 - 头部标题得高度
    height: calc( 100vh - 36px);
  }

  .recommend_wrap{
    // flex布局
    display: flex;
    flex-wrap: wrap;
    .recommend_item{
      width: 50%;
      border: 5rpx solid #fff;
    }
  };

  .moneths_wrap {
    .moneths_title {
      display: flex;
      justify-content: space-between;
      padding: 20rpx;
      .moneths_title_info {
        color: $color;
        font-size: 30rpx;
        font-weight: 600;
        display: flex;
        .moneths_info {
          .moneths_text {
            font-size: 36rpx;
          }
        }

        .moneths_text {
          font-size: 34rpx;
          color: #666;
          margin-left: 30rpx;
        }
        
    }

    .moneths_title_more {
      font-size: 24rpx;
      color: $color;
    }
  }
  .moneths_content {
    display: flex;
    flex-wrap: wrap;
    .moneths_item {
      width: 50%;
      border: 5rpx solid #fff;
    } 
  }

  
 
  };

  .hots_wrap {
    .hots_title {
      padding: 20rpx;
      text {
        border-left: 5rpx solid $color;
        padding-left: 20rpx;
        font-size: 34rpx;
        font-weight: 600;
      }
    }

    .hots_content {
      display: flex;
      flex-wrap: wrap;
      .hots_item {
        width: 33.33%;
        border: 5rpx solid #fff;
        image {

        }
      }
    }
  }
</style>
