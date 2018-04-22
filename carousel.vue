<template>
  <div class="carousel" :style=" 'height:' + imgHeight + 'px' " ref="carousel">
    <div class="carousel-imgs" :style=" 'transition-duration:' + transition + 'ms;transform:' + transform">
      <img class="img" :style="'margin: 0 ' + imgMargin + 'px;width:' + screenWidth + 'px;'" :src="item.url" v-for="(item, index) in list" :key="new Date().valueOf() + index">
    </div>
    <div class="carousel-dots">
      <span :class="['dot', index==currentIndex? 'active': '']" v-for="(item, index) in copyList" :key="new Date().valueOf() + index">
      </span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'carousel', // 无缝轮播组件
  data () {
    return {
      duration: 300, // 跳转时间
      imgMargin: 0,
      currentIndex: 0, // 当前所在slider
      transition: 0,
      transform: '',
      offsetLeft: 0,
      screenWidth: 0,
      copyList: [], // 点渲染数组
      list: [] // 图片渲染数组
    }
  },
  props: {
      imgWidth: {
        type: Number,
        default: 200
      },
      imgHeight: {
        type: Number,
        default: 200
      },
      imgList: {
        type: Array,
        default: []
      },
      autoTime: {  // 自动轮播时间
        type: Number,
        default: 2000
      }
  },
  watch: {
    offsetLeft: function (newval, val) {
      let result = Math.abs((newval / this.screenWidth) % this.copyList.length)
      if (!result) {
        this.currentIndex = this.copyList.length - 1
      } else {
        this.currentIndex = result - 1
      }
    },
    imgList: function (newval, val) {
      this.$nextTick(() => {
        if (newval.length > 0) {
          this.initCarousel(newval);
          this.initEvent();
          this.autoPlay();
        }
      })
    }
  },
  methods: {
    /*
     * 初始化轮播图,补节点
     */
    initCarousel (preList) {
      this.screenWidth = window.screen.width
      this.imgMargin = (this.screenWidth - this.imgWidth) / 2
      this.copyList = preList.slice(0); // 点渲染数组
      let list = preList.slice(0);
      this.trueWidth = list.length * this.screenWidth
      let last = list.pop();
      let first = list.shift();
      this.list = [last].concat(preList).concat([first]);// 渲染数组
      this.offsetLeft = -this.screenWidth // 偏移量
      this.transform = `translate3d(${this.offsetLeft}px, 0, 0)`
    },
    /*
     * 事件监听
     */
    initEvent () {
      this.touchStart && this.$refs.carousel.removeEventListener('touchstart', this.touchStart, false);
      this.touchMove && this.$refs.carousel.removeEventListener('touchmove', this.touchMove, false);
      this.touchEnd && this.$refs.carousel.removeEventListener('touchend', this.touchEnd, false);
      this.$refs.carousel.addEventListener('touchstart', this.touchStart ,false);
      this.$refs.carousel.addEventListener('touchmove', this.touchMove ,false);
      this.$refs.carousel.addEventListener('touchend', this.touchEnd ,false);
    },
    touchStart (e) {
      this.startX = e.touches[0].clientX;
      this.stopPlay()
    },
    touchMove (e) {
      this.moveDistance = e.touches[0].clientX - this.startX;
      this.moveX = this.offsetLeft + this.moveDistance
      // 边界问题
      if (this.moveX >= 0) { // 最左边
        this.moveX = -this.trueWidth + this.moveX
      }
      if (this.moveX <= -(this.list.length - 1) * this.screenWidth) {
        this.moveX = -1 * this.screenWidth + (this.moveX + (this.list.length - 1) * this.screenWidth)
      }
      this.transition = 0
      this.transform = `translate3d(${this.moveX}px, 0, 0)`
    },
    touchEnd (e) {
      this.offsetLeft = Math.round(this.moveX / this.screenWidth) * this.screenWidth
      this.transform = `translate3d(${this.offsetLeft}px, 0, 0)`
      this.transition = 300
      setTimeout(() => {
        if (this.transition) {
          this.transition = 0
        }
      }, this.transition)
      setTimeout(() => {
        this.autoPlay()
      }, this.transition + this.autoTime)
    },
    /*
     * 轮播
     */
    autoPlay () {
      this.timer = setInterval(() => {
        this.offsetLeft = this.offsetLeft - this.screenWidth // 默认方向
        this.transform = `translate3d(${this.offsetLeft}px, 0, 0)`
        this.transition = 300
        setTimeout(() => {
          if (this.transition) {
            this.transition = 0
            if (this.offsetLeft <= -(this.list.length - 1) * this.screenWidth) { // 边界
              this.offsetLeft = -1 * this.screenWidth
              this.transform = `translate3d(${this.offsetLeft}px, 0, 0)`
            }
          }
        }, this.transition)
      }, this.autoTime)
    },
    /*
     * 停止轮播
     */
    stopPlay () {
      clearInterval(this.timer);
    }
  }
}
</script>

<style lang="less" scoped>
.carousel{
  position: relative;
  overflow: hidden;
  .carousel-imgs{
    width: 100%;
    height: 100%;
    display: flex;
    .img{
    }
  }
  .carousel-dots{
    position: absolute;
    left: 50%;
    bottom: 20px;
    transform: translateX(-50%);
    .dot{
      display: inline-block;
      width: 5px;
      height: 5px;
      margin: 0 5px;
      background-color: #ccc;
      border-radius: 50%;
      &.active{
        background-color:red;
      }
    }
  }
}
</style>