<template>
  <view
    class="grid-lottery"
    :style="{
      height: gridLotteryHeight
    }"
  >
    <view class="lottery-list">
      <view
        v-for="(item, index) in formateList"
        :key="item.id"
        class="lottery-item"
        :style="{
          width: `${itemWidth}%`,
          height: `${itemHeight}${itemHeightUnit}`,
          transform: `translate(${place(index).x}%, ${place(index).y}${itemHeightUnit})`
        }"
      >
        <view class="mask" :style="{ opacity: index === currIndex ? 0 : 1 }"> </view>
        <image class="bg" :src="getStaticResourceUrl('user/product-background')" mode="aspectFit" />
        <view class="content-box">
          <image class="product-image" :src="item.goods.comMainPic" mode="aspectFit" />
          <view class="product-title-box">
            <view
              class="product-title u-line-2"
              :style="{
                fontSize: item.goods.comName.length > 20 ? '16rpx' : '18rpx'
              }"
            >
              {{ item.goods.comName }}
            </view>
          </view>
        </view>
      </view>
    </view>
    <view
      class="start-button"
      :style="{
        left: `${itemWidth}%`,
        right: `${itemWidth}%`,
        top: `${itemWidth}%`,
        bottom: `${itemWidth}%`
      }"
      @tap="onStart"
    >
      <slot name="start-button"></slot>
    </view>
    <view
      v-if="$slots['activity-box']"
      class="activity-box"
      :style="{
        width: `${itemWidth}%`,
        height: `${itemHeight}${itemHeightUnit}`,
        transform: `translate3d(${transform.x}%, ${transform.y}${itemHeightUnit}, 0)`,
        transition: `transform ${speed}ms`
      }"
    >
      <slot name="activity-box"></slot>
    </view>
  </view>
</template>

<script>
export default {
  name: 'grid-lottery',
  props: {
    // 转盘转动间隔
    speed: {
      type: Number,
      default: 50
    },
    // 是否逐渐减速
    isSpeedDecreases: {
      type: Boolean,
      default: true
    },
    // 减速开始时间
    speedDecreasesStartTime: {
      type: Number,
      default: 3000
    },
    // 最小速度
    minSpeed: {
      type: Number,
      default: 500
    },
    // 奖项外框高度
    itemHeight: {
      type: Number,
      default: 100
    },
    // 高度单位
    itemHeightUnit: {
      type: String,
      default: 'rpx'
    },
    // 总数必须为>=8 且被4整除
    list: {
      type: Array,
      default() {
        // return [
        //   {
        //     id: string,
        //     sort: number
        //   }
        // ];
        return []
      }
    }
  },
  data() {
    return {
      transform: {
        x: 0,
        y: 0
      },
      currIndex: 0,
      startFlag: false
    }
  },
  computed: {
    gridLotteryHeight({ itemHeight, formateList, colunmCount, itemHeightUnit }) {
      return `${itemHeight * ((formateList.length - colunmCount * 2) / 2 + 2)}${itemHeightUnit}`
    },
    itemWidth() {
      return 100 / this.colunmCount
    },
    colunmCount() {
      const len = this.list.length
      if (len > 8) {
        return 3 + Math.floor((len - 8) / 4)
      }
      return 3
    },

    formateList() {
      const len = this.list.length
      if (len > 7 && (len - 8) % 4 !== 0) {
        console.warn(
          '请检查数组，总数必须为>=8 且大余8部分部分能被4整除，如长度为8、12、16...的数组'
        )
      }
      const _list = this.list || []
      const formateList = _list.sort((a, b) => a.sort - b.sort)
      return formateList
    }
  },
  methods: {
    // 通过索引获取定位
    place(index) {
      const len = this.list.length
      // 序号
      const indexToOrder = index + 1
      if (indexToOrder > this.colunmCount) {
        // 去除上下行，总行数
        const rowCount = (len - this.colunmCount * 2) / 2
        // 非处于左侧
        if (index > this.colunmCount + rowCount) {
          // 处于右侧
          if (indexToOrder > this.colunmCount * 2 + rowCount) {
            return {
              x: 0,
              y: this.itemHeight * (rowCount * 2 + this.colunmCount * 2 - index)
            }
          }
          // 处于底部
          return {
            x: 100 * (this.colunmCount * 2 + rowCount - indexToOrder),
            y: this.itemHeight * (rowCount + 1)
          }
        }
        // 处于右侧
        return {
          x: 100 * (this.colunmCount - 1),
          y: this.itemHeight * (indexToOrder - this.colunmCount)
        }
      }
      // 处于顶部
      return {
        x: 100 * index,
        y: 0
      }
    },
    /**
     * 开始抽奖
     * @param {string|number} id 获奖id
     * @param {number} minTime 转盘最小持续时间
     */
    async start(id, minTime) {
      if (this.startFlag) return
      const len = this.formateList.length
      const activeIndex = this.formateList.findIndex((item) => item.id === id)
      if (activeIndex === -1) {
        const errorText = `没有找到该中奖id ${id}`
        console.error(errorText)
        return this.$emit('onError', errorText)
      }
      let time = minTime
      // 动画速度
      let speed = this.speed
      const startRotate = () => {
        this.startFlag = true
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          // 抽奖结束
          if (time < 1 && this.currIndex === activeIndex) {
            this.startFlag = false
            clearTimeout(this.timer)
            return this.$emit('onEnd', this.formateList[activeIndex])
          }
          time -= speed
          if (time < 3000 && this.isSpeedDecreases) {
            // 调整动画速度
            speed = speed >= this.minSpeed ? this.minSpeed : speed + 20
          }
          this.currIndex = this.currIndex >= len - 1 ? 0 : this.currIndex + 1
          this.transform = this.place(this.currIndex)
          return startRotate()
        }, speed)
      }
      startRotate()
    },
    onStart() {
      this.$emit('onStart')
    }
  }
}
</script>

<style lang="scss" scoped>
.grid-lottery {
  position: relative;

  .start-button {
    position: absolute;
  }

  .lottery-list {
    display: flex;
    flex-wrap: wrap;

    .lottery-item {
      position: absolute;
      left: 0;
      top: 0;
    }
  }

  .activity-box {
    position: absolute;
    top: 0;
    left: 0;
  }
}

.lottery-item {
  color: #1f2638;
  height: 180rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  position: relative;
  .mask {
    position: absolute;
    top: 6rpx;
    left: 6rpx;
    bottom: 6rpx;
    right: 6rpx;
    background-color: rgba(0, 0, 0, 0.4);
    z-index: 3;
    border-radius: 12rpx;
  }

  .bg {
    width: 100%;
    height: 100%;
    position: absolute;
    z-index: 1;
  }

  .content-box {
    position: relative;
    z-index: 2;
    text-align: center;
    padding-top: 10rpx;

    .product-image {
      width: 108rpx;
      height: 70rpx;
    }

    .product-title-box {
      display: flex;
      align-items: center;
      justify-content: center;
      height: calc(2em + 8rpx);
      font-size: 18rpx;
    }

    .product-title {
      color: #1f2638;
      line-height: 22rpx;
      padding: 0 15rpx;
    }
  }
}
</style>
