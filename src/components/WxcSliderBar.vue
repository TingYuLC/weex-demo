<template>
  <div
    ref="bar-container"
    class="slider-bar-container"
    :style="containerStyle">
    <div
      class="range-bar"
      :style="rangeBarStyle">
        <div
          ref="value-bar"
          class="value-bar"
          :style="valueBarStyle">
        </div>
    </div>
    <div
      v-if="isIOS"
      ref="slide-block-1"
      class="slide-block"
      @horizontalpan="weexHandler"
      :prevent-move-event="true"
      :style="blockStyle1">
    </div>
    <div
      v-else
      ref="slide-block-1"
      class="slide-block"
      @panstart="weexStartHandler"
      @panmove="weexMoveHandler"
      :prevent-move-event="true"
      :style="blockStyle1">
    </div>
  </div>
</template>
<script>
import Binding from 'weex-bindingx/lib/index.weex.js'
import { Utils, BindEnv } from 'weex-ui'

const animation = weex.requireModule('animation')
const dom = weex.requireModule('dom')

export default {
  data () {
    return {
      diffX1: 0,
      startX: 0,
      startDiffX1: 0,
      timeout: 100,
      DPR: 1,
      isAndroid: Utils.env.isAndroid(),
      isWeb: Utils.env.isWeb(),
      isIOS: Utils.env.isIOS()
    }
  },
  props: {
    length: {
      type: Number,
      default: 500
    },
    value: {
      type: Number,
      default: 0
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 100
    },
    height: {
      type: Number,
      default: 4
    },    
    invalidColor: {
      type: String,
      default: '#E0E0E0'
    },
    validColor: {
      type: String,
      default: '#EE9900'
    },
  },
  created () {
    if (Utils.env.isWeb()) {
      this.env = 'web'
      this.DPR = window.devicePixelRatio ? window.devicePixelRatio : 1
    } else {
      this.DPR = weex.config.env.scale
    }
  },
  mounted () {
    this.block1 = this.$refs['slide-block-1'];        // 左侧滑块
    this.valueBar = this.$refs['value-bar'];          // 黄色值条
    this.barContainer = this.$refs['bar-container'];  // 滚动条容器
    this.diffX1 = this._getDiffX(this.value);
    // 是否支持expresstionBinding
    if (BindEnv.supportsEB() && Binding.prepare) {
      this.block1 && Binding.prepare({
        anchor: this.block1.ref,
        eventType: 'pan'
      })
      this.valueBar && Binding.prepare({
        anchor: this.valueBar.ref,
        eventType: 'pan'
      })
    }
    // 由于weex在mounted后渲染是异步的不能确保元素渲染完成，需要异步执行
    setTimeout(() => {
      dom.getComponentRect(this.barContainer, option => {
        const { left } = option.size
        this.leftDiffX = left
      })
    }, 100)
  },
  computed: {
    containerStyle () {
      return {
        width: `${this.length + 56}px`,
        height: '56px'
      }
    },
    rangeBarStyle () {
      return {
        width: `${this.length}px`,
        height: `${this.height}px`,
        flexDirection: 'row',
        backgroundColor: this.invalidColor
      }
    },
    valueBarStyle () {
      let left = this.diffX1 - this.length
      let width = this.length;
      return {
        width: width + 'px',
        height: this.height + 'px',
        transform: `translateX(${left}px)`,
        backgroundColor: this.validColor
      }
    },
    blockStyle1 () {
      let left = this.diffX1
      return {
        transform: `translateX(${left}px)`
      }
    }
  },
  watch: {
    value (e) {
      this.diffX1 = this._getDiffX(e);
    }
  },
  methods: {
    weexHandler (e) {
      const self = this
      switch (e.state) {
        case 'start':
          self.bindBlock1()
          break
        case 'move':
          dom.getComponentRect(this.block1, option => {
            const { left } = option.size
            const value = this._getValue(left - this.leftDiffX)
            this.$emit('updateValue', value)
          });
          break
        case 'end':
          break
        default:
          break
      }
    },
    weexStartHandler (e) {
      if (this.isWeb) {
        this.webStartHandler(e)
      } else if (this.isAndroid) {
        this.AndroidStartHandle(e)
      }
    },
    weexMoveHandler (e) {
      if (this.isWeb) {
        this.webMoveHandler(e)
      } else if (this.isAndroid) {
        this.AndroidMoveHandle(e)
      }
    },
    webStartHandler(e) {
      this.startX = e.touch.clientX
      this.startDiffX1 = this.diffX1
    },
    webMoveHandler(e) {
      const deltaX = (e.touch.clientX - this.startX) * this.DPR
      const diff = this.startDiffX1 + deltaX
      if (diff >= 0 && diff <= this.length) {
        this.diffX1 = diff
      } else if (diff > this.length) {
        this.diffX1 = this.length
      } else if (diff < 0) {
        this.diffX1 = 0
      }
      this.$emit('updateValue', this._getValue(this.diffX1))
    },
    AndroidStartHandle (e) {
      this.startX = e.changedTouches[0].screenX
      this.startDiffX1 = this.diffX1
    },
    AndroidMoveHandle (e) {
      const deltaX = (e.changedTouches[0].screenX - this.startX)
      const diff = this.startDiffX1 + deltaX

      if (diff >= 0 && diff <=this.length) {
        this.diffX1 = diff
      } else if (diff < 0) {
        this.diffX1 = 0
      } else {
        this.diffX1 = this.length
      }
      this.$emit('updateValue', this._getValue(this.diffX1));     
    },  
    bindBlock1() {
      if (!this.isIOS) return
      const self = this
      // 初始化按钮&条的大小范围
      let blockMax1 = 0
      blockMax1 = self.length
      const startLeft = self.diffX1 - blockMax1
      const props = [{
        element: self.block1.ref,
        property: 'transform.translateX',
        expression: `min(${blockMax1}, max(x + ${self.diffX1}, 0))`
      }, {
        element: self.valueBar.ref,
        property: 'transform.translateX',
        expression: `min(0, max(x + ${startLeft}, -${blockMax1}))`
      }]
      const gesTokenObj = Binding.bind({
        anchor: self.block1.ref,
        eventType: 'pan',
        props
      }, (e) => {
        if (e.state === 'end' || e.state === 'cancel' || e.state === 'exit') {
          // 限制diffX1范围
          self.diffX1 = self._restrictValue(self.diffX1 + e.deltaX)
          self.bindBlock1()
        }
      })
      this.gesToken1 = gesTokenObj.token
    },
    // 限制取值范围
    _restrictValue(value) {
      if (value < 0) {
        return 0
      } else if (value > this.length) {
        return this.length
      } else {
        return value
      }
    },
    // 根据x方向偏移量计算value
    _getValue (diffX) {
      return Math.round((diffX / this.length) * (this.max - this.min) + this.min);
    },
    // 根据value和length计算x方向偏移值
    _getDiffX (value) {
      return ((value - this.min) / (this.max - this.min)) * this.length;
    }
  }
}
</script>
<style scoped>
.slider-bar-container {
  height: 56px;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}
.range-bar{
  overflow: hidden;
}
.value-bar {
  height: 4px;
  overflow: hidden;
}
.slide-block {
  width: 56px;
  height: 56px;
  background-color: #ffffff;
  border-radius: 28px;
  border-width: 1px;
  border-color: rgba(0, 0, 0, 0.1);
  position: absolute;
  left: 0;
  bottom: 0;
}
</style>
