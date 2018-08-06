<template>
  <div ref="wrap">
    <div ref="realBox" :style="pos" @mouseenter="enter" @mouseleave="leave">
      <div ref="slotList" class="overflow">
        <slot></slot>
      </div>
      <div v-html="copyHtml" class="overflow"></div>
    </div>
  </div>
</template>
<script>
  const defaultOption={
    step: 1, //步长
    limitMoveNum: 5, //启动无缝滚动最小数据数
    hoverStop: true, //是否启用鼠标hover控制
    direction: 1, // 0 往下 1 往上
    openTouch: true, //开启移动端touch
    singleHeight: 0, //单条数据高度有值hoverStop关闭
    singleWidth: 0, //单条数据宽度有值hoverStop关闭
    waitTime: 1000, //单步停止等待时间
    autoPlay: true,
    switchDelay: 400,
    switchDisabledClass: 'disabled',
    isSingleRemUnit: false // singleWidth/singleHeight 是否开启rem度量
  }
  export default {
    name: 'vue-text-scroll',
    data () {
      return {
        xPos: 0,
        yPos: 0,
        delay: 0,
        copyHtml: '',
        height: 0,
        width: 0, // 外容器宽度
        realBoxWidth: 0, // 内容实际宽度
        reqFrame: null, // move动画的animationFrame定时器
        singleWaitTime: null, // single 单步滚动的定时器
        isHover: false, // mouseenter mouseleave 控制this._move()的开关
      }
    },
    props: {
      data: {
        type: Array,
        default: () => {
          return []
        }
      },
      option: {
        type: Object,
        default: () => {
          return {}
        }
      }
    },
    computed: {
      options () {
        return  Object.assign({},defaultOption, this.option)
      },
      pos () {
        return {
          transform: `translate(${this.xPos}px,${this.yPos}px)`,
          transition: `all ${this.ease || 'ease-in'} ${this.delay}ms`,
          overflow: 'hidden'
        }
      },
      moveSwitch () {
        return this.data.length < this.options.limitMoveNum
      },
      hoverStop () {
        return !this.options.autoPlay || !this.options.hoverStop || this.moveSwitch
      },
      baseFontSize () {
        return this.options.isSingleRemUnit ? parseInt(window.getComputedStyle(document.documentElement, null).fontSize) : 1
      },
      realSingleStopWidth () {
        return this.options.singleWidth * this.baseFontSize
      },
      realSingleStopHeight () {
        return this.options.singleHeight * this.baseFontSize
      }
    },
    methods: {
      _cancle () {
        cancelAnimationFrame(this.reqFrame || '')
      },
      enter () {
        if (this.hoverStop) return
        this.isHover = true //关闭_move
        // 防止蛋疼的人频频hover进出单步滚动 导致定时器乱掉
        if (this.singleWaitTime) clearTimeout(this.singleWaitTime)
        this._cancle()
      },
      leave () {
        if (this.hoverStop) return
        this.isHover = false //开启_move
        this._move()
      },
      _move () {
        // 鼠标移入时拦截_move()
        if (this.isHover) return
        this._cancle() //进入move立即先清除动画 防止频繁touchMove导致多动画同时进行
        this.reqFrame = requestAnimationFrame(
          function () {
            if (!this.$refs.realBox) return //fixed 路由之间切换报this.$refs.realBox.offsetHeigh undefined bug
            const h = this.$refs.realBox.offsetHeight / 2  //实际高度
            const w = this.$refs.slotList.offsetWidth //宽度
            const direction = this.options.direction //滚动方向
            if (direction === 1) { // 上
              if (Math.abs(this.yPos) >= h) this.yPos = 0
              this.yPos -= this.options.step
            } else if (direction === 0) { // 下
              if (this.yPos >= 0) this.yPos = h * -1
              this.yPos += this.options.step
            }
            if (this.singleWaitTime) clearTimeout(this.singleWaitTime)
            if (!!this.realSingleStopHeight) { //是否启动了单行暂停配置
              if (Math.abs(this.yPos) % this.realSingleStopHeight < 1) { // 符合条件暂停waitTime
                this.singleWaitTime = setTimeout(() => {
                  this._move()
                }, this.options.waitTime)
              } else {
                this._move()
              }
            } else if (!!this.realSingleStopWidth) {
              if (Math.abs(this.xPos) % this.realSingleStopWidth < 1) { // 符合条件暂停waitTime
                this.singleWaitTime = setTimeout(() => {
                  this._move()
                }, this.options.waitTime)
              } else {
                this._move()
              }
            } else {
              this._move()
            }
          }.bind(this)
        )
      },
      _initMove () {
        this.$nextTick(() => {
          this.height = this.$refs.wrap.offsetHeight
          this.width = this.$refs.wrap.offsetWidth

          if (!this.options.autoPlay) {
            this.ease = 'linear'
            this.delay = this.options.switchDelay
            return
          }
          this._dataWarm(this.data)
          this.copyHtml = '' //清空copy
          // 是否可以滚动判断
          if (this.moveSwitch) {
            this._cancle()
            this.yPos = this.xPos = 0
          } else {
            let timer
            if (timer) clearTimeout(timer)
            this.copyHtml = this.$refs.slotList.innerHTML
            this._move()
          }
        })
      },
      _dataWarm (data) {
        if (data.length > 100) {
          console.warn(`数据达到了${data.length}条有点多哦~,可能会造成部分老旧浏览器卡顿。`);
        }
      },
      arrayEqual(arr1, arr2) {
        if (arr1 === arr2) return true;
        if (arr1.length !== arr2.length) return false;
        for (var i = 0; i < arr1.length; ++i) {
          if (arr1[i] !== arr2[i]) return false;
        }
        return true;
      }
    },
    mounted () {
      this._initMove()
    },
    watch: {
      data (newData, oldData) {
        this._dataWarm(newData)
        //监听data是否有变更
        if (!this.arrayEqual(newData, oldData)) {
          this._cancle()
          this._initMove()
        }
      }
    },
    beforeDestroy () {
      this._cancle()
    }
  }
</script>
<style scoped>
  .overflow{
    overflow: hidden;
  }
</style>
