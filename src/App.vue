<template>
  <div id="app">
    <Video class="video" controls v-bind.sync="videoParams">
      <source src="/1.mp4"/>
    </Video>
  </div>
</template>

<script>

import Video from './Video'
export default {
  name: 'app',
  data () {
    return {
      videoParams: {
        volume: null,
        currentTime: null,
        playbackRate: null,
        pause: false,
        playing: false
      }
    }
  },
  components: {
    Video
  }
}
</script>

<style lang="less">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
  .video {
    width: 640px;
    height: 480px;
  }
</style>
