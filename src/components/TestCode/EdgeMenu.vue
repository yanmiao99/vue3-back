<!--
按照目前的情况下, 好像只是适合移动端
-->
<template>
  <transition>
    <div
        ref="dragIcon"
        class="dragIcon not-select"
        @click="onBtnClicked"
        @touchstart.stop="handleTouchStart"
        @touchmove.prevent.stop="handleTouchMove($event)"
        @touchend.stop="handleTouchEnd"
        :style="{left: left + 'px',top: top + 'px',width: itemWidth + 'px',height: itemHeight + 'px'}"
        v-if="isShow">
      Menu
    </div>
  </transition>
  <!--@touchstart.stop="handleTouchStart"-->
  <!--@touchmove.prevent.stop="handleTouchMove($event)"-->
  <!--@touchend.stop="handleTouchEnd"-->

  <!--@mousedown.stop="handleTouchStart"-->
  <!--@mousemove.prevent.stop="handleTouchMove($event)"-->
  <!--@mouseup.stop="handleTouchEnd"-->



</template>

<script>

export default {
  props: {
    itemWidth: {
      type: Number,
      default: 50,
    },
    itemHeight: {
      type: Number,
      default: 50,
    },
  },
  data() {
    return {
      left: 0,
      top: 0,
      startToMove: false,
      isShow: true,
      timer: null,
      currentTop: null,
      clientW: document.documentElement.clientWidth, //视口宽
      clientH: document.documentElement.clientHeight, //视口高
    }
  },
  created() {
    this.left = this.clientW - this.itemWidth - 15
    this.top = this.clientH / 2 - this.itemHeight / 2
  },
  mounted() {
    this.bindScrollEvent()
  },
  beforeDestroy() {
    // 记得销毁一些全局的的事件
    this.removeScrollEvent()
  },
  methods: {
    onBtnClicked() {
      this.$emit('onFloatBtnClicked')
    },
    handleTouchStart() {
      this.startToMove = true
      this.$refs.dragIcon.style.transition = 'none'
    },
    handleTouchMove(e) {
      const clientX = e.targetTouches[0].clientX //手指相对视口的x
      const clientY = e.targetTouches[0].clientY //手指相对视口的y
      const isInScreen =
          clientX <= this.clientW &&
          clientX >= 0 &&
          clientY <= this.clientH &&
          clientY >= 0
      if (this.startToMove && e.targetTouches.length === 1) {
        if (isInScreen) {
          this.left = clientX - this.itemWidth / 2
          this.top = clientY - this.itemHeight / 2
        }
      }
    },
    handleTouchEnd() {
      if (this.left < this.clientW / 2) {
        this.left = 15 //不让贴边 所以设置15没设置0
        this.handleIconY()
      } else {
        this.left = this.clientW - this.itemWidth - 15 //不让贴边 所以减15
        this.handleIconY()
      }
      this.$refs.dragIcon.style.transition = 'all .3s'
    },
    handleIconY() {
      if (this.top < 0) {
        this.top = 15 //不上帖上边所以设置为15 没设置0
      } else if (this.top + this.itemHeight > this.clientH) {
        this.top = this.clientH - this.itemHeight - 15 //不让帖下边所以减15
      }
    },
    bindScrollEvent() {
      window.addEventListener('scroll', this.handleScrollStart)
    },
    removeScrollEvent() {
      window.removeEventListener('scroll', this.handleScrollStart)
    },
    handleScrollStart() {
      this.isShow = false
      this.timer && clearTimeout(this.timer)
      this.timer = setTimeout(() => {
        this.handleScrollEnd()
      }, 300)
      this.currentTop =
          document.documentElement.scrollTop || document.body.scrollTop
    },
    handleScrollEnd() {
      this.scrollTop =
          document.documentElement.scrollTop || document.body.scrollTop
      // 判断是否停止滚动的条件
      if (this.scrollTop == this.currentTop) {
        this.isShow = true
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.dragIcon {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  position: fixed;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #2270fe; /* 浏览器不支持的时候显示 */
  color: #fff;
  word-wrap: break-word;
  word-break: break-all;
  font-size: 12px;

}

.v-enter {
  opacity: 1;
}

.v-leave-to {
  opacity: 0;
}

.v-enter-active,
.v-leave-active {
  transition: opacity 0.3s;
}

.not-select {
  -moz-user-select: none; /*火狐*/
  -webkit-user-select: none; /*webkit浏览器*/
  -ms-user-select: none; /*IE10*/
  -khtml-user-select: none; /*早期浏览器*/
  user-select: none;
}

.drag-item {
  position: absolute;
  width: 100px;
  height: 100px;
  background-color: deepskyblue;
  transform: scale(1);
}

.v-drag-active {
  transform: scale(1.2);
}

.v-drag-inactive {
  transform: scale(1);
}
</style>
