<template>
  <div class="wrapper">
    <text>{{value}}</text>
    <wxc-slider-bar @updateValue="updateValue" :min="0" :max="20000" :value="8000" :length="644" />
  </div>
</template>

<script>
import WxcSliderBar from './components/WxcSliderBar'
export default {
  name: 'App',
  components: {
    WxcSliderBar
  },
  data () {
    return {
      value: 8000
    }
  },
  methods: {
    updateValue (value) {
      this.value = value
    }
  }
}
</script>

<style scoped>
  .wrapper {
    flex: 1;
    justify-content: center;
    align-items: center;
  }
  .logo {
    width: 424px;
    height: 200px;
  }
  .greeting {
    text-align: center;
    margin-top: 70px;
    font-size: 50px;
    color: #41B883;
  }
  .message {
    margin: 30px;
    font-size: 32px;
    color: #727272;
  }
</style>
