<template>
  <view
    class="grid-lottery"
    :style="{
      height: `${itemHeight * ((formateList.length - colunmCount * 2) / 2 + 2)}${itemHeightUnit}`
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
        <slot name="lottery-item" :test="item"></slot>
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
    // 奖项外框高度
    itemHeight: {
      type: Number,
      default: 100
    },
    // 高度单位
    itemHeightUnit: { type: String, default: 'rpx' },
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
        return [];
      }
    }
  },
  data() {
    return {
      transform: {
        x: 0,
        y: 0
      },
      speed: 50,
      currIndex: 0,
      startFlag: false
    };
  },
  computed: {
    itemWidth() {
      return 100 / this.colunmCount;
    },
    colunmCount() {
      const len = this.list.length;
      if (len > 8) {
        return 3 + Math.floor((len - 8) / 4);
      }
      return 3;
    },

    formateList() {
      const len = this.list.length;
      if (len > 7 && (len - 8) % 4 !== 0) {
        console.warn('请检查数组，总数必须为>=8 且大余8部分部分能被4整除，如长度为8、12、16...的数组');
      }
      const _list = this.list || [];
      const formateList = _list.sort((a, b) => a.sort - b.sort);
      return formateList;
    }
  },
  methods: {
    // 通过索引获取定位
    place(index) {
      const len = this.list.length;
      // 序号
      const indexToOrder = index + 1;
      if (indexToOrder > this.colunmCount) {
        // 去除上下行，总行数
        const rowCount = (len - this.colunmCount * 2) / 2;
        // 非处于左侧
        if (index > this.colunmCount + rowCount) {
          // 处于右侧
          if (indexToOrder > this.colunmCount * 2 + rowCount) {
            return {
              x: 0,
              y: this.itemHeight * (rowCount * 2 + this.colunmCount * 2 - index)
            };
          }
          // 处于底部
          return {
            x: 100 * (this.colunmCount * 2 + rowCount - indexToOrder),
            y: this.itemHeight * (rowCount + 1)
          };
        }
        // 处于右侧
        return {
          x: 100 * (this.colunmCount - 1),
          y: this.itemHeight * (indexToOrder - this.colunmCount)
        };
      }
      // 处于顶部
      return { x: 100 * index, y: 0 };
    },
    /**
     * 开始抽奖
     * @param {string|number} id 获奖id
     * @param {number} minTime 转盘最小持续时间
     */
    async start(id, minTime) {
      if (this.startFlag) return;
      const len = this.formateList.length;
      const activeIndex = this.formateList.findIndex(item => item.id === id);
      if (activeIndex === -1) {
        const errorText = `没有找到该中奖id ${id}`;
        console.error(errorText);
        return this.$emit('onError', errorText);
      }
      let time = minTime;
      // 动画速度
      this.speed = 50;
      const startRotate = () => {
        this.startFlag = true;
        clearTimeout(this.timer);
        this.timer = setTimeout(() => {
          // 抽奖结束
          if (time < 1 && this.currIndex === activeIndex) {
            this.startFlag = false;
            clearTimeout(this.timer);
            return this.$emit('onEnd', this.formateList[activeIndex]);
          }
          time -= this.speed;
          if (time < 5000) {
            // 调整动画速度
            this.speed = this.speed >= 1000 ? 1000 : this.speed + 20;
          }
          this.currIndex = this.currIndex >= len - 1 ? 0 : this.currIndex + 1;
          this.transform = this.place(this.currIndex);
          return startRotate();
        }, this.speed);
      };
      startRotate();
    },
    onStart() {
      this.$emit('onStart');
    }
  }
};
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
</style>
