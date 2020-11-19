<template>
  <view class="content">
    <grid-lottery itemHeight="200" ref="gridLotteryRef" style="width: 350px;" :list="list" @onStart="onStart" @onEnd="onEnd">
      <!-- <view slot="lottery-item" slot-scope="slotProps">{{ slotProps }}</view> -->
      <template slot="lottery-item" slot-scope="{ test }">
        <view class="lottery-item">
          <view>{{ test.sort }} - {{ test.id }}</view>
          <view>{{ test.title }}</view>
        </view>
      </template>
      <view slot="start-button" class="start-button">开始</view>
      <view slot="activity-box" class="activity-box"></view>
    </grid-lottery>
  </view>
</template>

<script>
import gridLottery from '@/components/grid-lottery.vue';
export default {
  components: {
    gridLottery
  },
  data() {
    return {
      list: [
        {
          id: 1,
          sort: 0,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 2,
          sort: 1,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 3,
          sort: 2,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 4,
          sort: 3,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 5,
          sort: 4,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 6,
          sort: 5,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 7,
          sort: 6,
          title: '\u6BDB\u5DFE'
        },
        {
          id: 8,
          sort: 7,
          title: '\u6BDB\u5DFE'
        }, {
          id: 8,
          sort: 7,
          title: '\u6BDB\u5DFE'
        }, {
          id: 8,
          sort: 7,
          title: '\u6BDB\u5DFE'
        }, {
          id: 8,
          sort: 7,
          title: '\u6BDB\u5DFE'
        }, {
          id: 8,
          sort: 7,
          title: '\u6BDB\u5DFE'
        }
      ]
    };
  },
  onLoad() {},
  methods: {
    onStart() {
      this.$refs.gridLotteryRef.start(3, 10000);
    },
    onEnd(item) {
      console.log(item);
    }
  }
};
</script>

<style scoped>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.lottery-item {
  background-color: orange;
  color: #fff;
  height: 200rpx;
  border: 1px solid #ededed;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
.activity-box {
  border: 1px solid red;
  width: 100%;
  height: 200rpx;
}
.start-button {
  background-color: #2c405a;
  color: #fff;
  text-align: center;
  font-size: 30rpx;
  line-height: 400rpx;
  font-weight: bold;
}
</style>
