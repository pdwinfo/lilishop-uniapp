<template>
  <view class="wrapper">
    <view class="tips">我的投诉信息</view>
    <u-cell-group>
      <u-cell-item :arrow="false" :value="complainDetail.goodsName" title="投诉商品"></u-cell-item>
      <u-cell-item :arrow="false" :value="statusData[complainDetail.complainStatus]" title="投诉状态"></u-cell-item>
      <u-cell-item :arrow="false" :value="complainDetail.createTime" title="投诉时间"></u-cell-item>
      <u-cell-item :arrow="false" :value="complainDetail.complainTopic" title="投诉主题"></u-cell-item>
      <u-cell-item :arrow="false" :value="complainDetail.content" title="投诉内容"></u-cell-item>
      <view class="row" v-if="complainDetail.orderComplaintImages">
        <u-image width="100rpx" height="100rpx" border-radius="10" style="margin: 0 10rpx" v-for="(item, index) in complainDetail.orderComplaintImages" :key="index" :src="item"
          @click="preview(complainDetail.orderComplaintImages, index)" />
      </view>
    </u-cell-group>
    <view class="tips">商家申诉信息</view>
    <u-cell-group>
      <u-cell-item :arrow="false" :value="complainDetail.appealTime || '暂无'" title="申诉时间"></u-cell-item>
      <u-cell-item :arrow="false" :value="complainDetail.appealContent || '暂无'" title="申诉内容"></u-cell-item>
      <view class="row" v-if="complainDetail.appealImagesList">
        <u-image width="100rpx" height="100rpx" border-radius="10" style="margin: 0 10rpx" v-for="(item, index) in complainDetail.appealImagesList"
          @click="preview(complainDetail.appealImagesList, index)" :key="index" :src="item" />
      </view>
    </u-cell-group>
    <view class="tips">对话详情</view>
    <view class="speak-way" v-if="complainDetail.orderComplaintCommunications">
      <view class="speak-msg seller" :key="i" v-for="(complaint, i) in complainDetail.orderComplaintCommunications">
        {{
          complaint.owner == "PLATFORM"
            ? "平台"
            : complaint.owner == "BUYER"
            ? "买家"
            : "卖家"
        }}：
        <span>{{ complaint.content }}</span>
      </view>
    </view>
    <view class="speak-way" v-else>暂无对话</view>
	<div v-if="complainDetail.complainStatus!='COMPLETE'">
		<view class="tips">回复对话</view>
		<view class="cell-item complain-content">
		  <view class="cell-view content">
			<u-input type="textarea" height="70rpx" auto-height v-model="complainValue" />
		  </view>
		</view>
		<view class="submit-btn" @click="handleSubmit">回复</view>
	</div>
    <view class="tips">平台仲裁</view>
    <u-cell-group>
      <u-cell-item :arrow="false" title="仲裁意见" :value="complainDetail.arbitrationResult || '暂无'"></u-cell-item>
    </u-cell-group>
  </view>
</template>

<script>
import { getComplainDetail, communication } from "@/api/after-sale";
export default {
  data() {
    return {
      complainId: "",
      complainValue: "", //回复内容
      complainDetail: "", //投诉详情
      statusData: {
        NEW: "新投诉",
        NO_APPLY: "未申请",
        APPLYING: "申请中",
        COMPLETE: "已完成",
        EXPIRED: "已失效",
        CANCEL: "已取消",
        WAIT_ARBITRATION:"等待仲裁"
      },
    };
  },

  onLoad(option) {
    this.complainId = option.id;
    this.init(option.id);
  },
  methods: {
    /**
     * 点击图片放大或保存
     */
    preview(urls, index) {
      uni.previewImage({
        current: index,
        urls: urls,
        longPressActions: {
          itemList: ["保存图片"],
          success: function (data) {},
          fail: function (err) {},
        },
      });
    },
    handleSubmit() {
      if (!this.complainValue) {
        uni.showToast({
          title: "请输入回复内容",
          duration: 2000,
          icon: "none",
        });
        return;
      }
      let params = {
        content: this.complainValue,
        complainId: this.complainId,
      };
      communication(params).then((res) => {
        if (res.data.success) {
          uni.showToast({
            title: "回复成功",
            duration: 2000,
            icon: "none",
          });
          this.complainValue = '';
          this.init(this.complainId);
        } else {
          uni.showToast({
            title: res.data.message,
            duration: 2000,
            icon: "none",
          });
        }
      });

    },
    /**
     * 初始化投诉详情
     */
    init(id) {
      uni.showLoading({
        title: "加载中",
      });
      getComplainDetail(id).then((res) => {
        if (res.data.success) {
          this.complainDetail = res.data.result;
        } else {
          uni.showToast({
            title: res.data.message,
            duration: 2000,
            icon: "none",
          });
        }
         if (this.$store.state.isShowToast){ uni.hideLoading() };
      });
    },
  },
};
</script>
<style lang="scss" scoped>
.row {
  display: flex;
  flex-wrap: wrap;
  padding: 26rpx 32rpx;
}

.speak-msg {
  padding: 26rpx 32rpx;
  > span {
    color: #999 !important;
  }
}
.admin {
  color: $main-color;
}
.speak-way {
  background: #fff;
}
.wrapper {
  padding: 16rpx;
}
.tips {
  margin: 40rpx 32rpx;
}
.cell {
  width: 100%;
  background: #fff;
  padding: 26rpx;
}
.complain-content {
  margin: 40rpx 32rpx;
  display: flex;
  align-items: center;

  .content {
    width: 100%;
  }
  .title {
    width: 140rpx;
  }
}
.submit-btn {
  width: 70%;
  margin: 0 auto;
  height: 80rpx;
  line-height: 80rpx;
  color: #fff;
  text-align: center;
  background: $light-color;
  margin-top: 20px;
  border-radius: 200px;
}
</style>
