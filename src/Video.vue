<template>
  <div class="vue-video">
    <video ref="video"
           :src="sourceType === 'source' ? videoSrc : undefined"
           :controls="!customizeControls && controls"
           :autoplay="autoplay"
           @abort="onAbort"
           @canplay="onCanPlay"
           @canplaythrough="onCanPlayThrough"
           @durationchange="onDurationChange"
           @emptied="onEmptied"
           @ended="onEnded"
           @error="onError"
           @loadeddata="onLoadedData"
           @loadedmetadata="onLoadedMetaData"
           @loadstart="onLoadStart"
           @pause="onPause"
           @play="onPlay"
           @playing="onPlaying"
           @progress="onProgress"
           @ratechange="onRateChange"
           @seeked="onSeeked"
           @seeking="onSeeking"
           @stalled="onStalled"
           @suspend="onSuspend"
           @timeupdate="onTimeUpdate"
           @volumechange="onVolumeChange"
           @waiting="onWaiting"
    >
      <slot>
        <template v-if="sourceType === 'source-list'">
          <source v-for="source in src" :src="source.src" :type="source.type" :key="source.src">
          Your browser does not support this video.
        </template>
      </slot>
    </video>
    <slot v-if="customizeMask" name="mask"
          :duration="duration"
          :seeking="seeking"
          :playing="playing"
          :loading="loading"
          :loading-meta-data="loadingMetaData"
          :can-play="canPlay"
          :can-play-through="canPlayThrough"
          :pause="pause"
          :current-time="currentTime"
          :volume="volume"
          :playbackRate="playbackRate"
          :muted="muted"
    >
      <div class="vue-video-mask">
        遮罩样式暂未实现
      </div>
    </slot>
    <slot v-if="customizeControls && controls" name="controls"
          :duration="duration"
          :seeking="seeking"
          :playing="playing"
          :loading="loading"
          :loading-meta-data="loadingMetaData"
          :can-play="canPlay"
          :can-play-through="canPlayThrough"
          :pause="pause"
          :current-time="currentTime"
          :volume="volume"
          :playbackRate="playbackRate"
          :muted="muted"
    >
      <div class="vue-video-controls">
        控制栏样式暂未实现
      </div>
    </slot>
  </div>
</template>

<script>

const extensions = []

function findExtension (name) {
  for (const extension of extensions) {
    if (extension.type instanceof RegExp) {
      if (extension.type.test(name)) {
        return extension
      } else {
        continue
      }
    }
    if (extension.type === name) {
      return extension
    }
  }
}

export default {
  name: 'Video',
  props: {
    src: {
      type: [String, Array]
    },
    pause: Boolean,
    currentTime: Number,
    volume: Number,
    playbackRate: Number,
    muted: Boolean,
    customizeControls: Boolean,
    customizeMask: Boolean,
    controls: Boolean,
    autoplay: Boolean,
    isLive: Boolean,
    type: String
  },
  data () {
    return {
      videoSrc: this.src,
      duration: -1,
      seeking: false,
      playing: false,
      loading: false,
      loadingMetaData: false,
      canPlay: false,
      canPlayThrough: false
    }
  },
  computed: {
    sourceType () {
      if (Array.isArray(this.videoSrc)) {
        return 'source-list'
      } else if (typeof this.videoSrc === 'string') {
        return 'source'
      } else {
        return 'unknown'
      }
    },
    video () {
      window.v = this.$refs.video
      return this.$refs.video
    },
    currentSrc () {
      return this.video.currentSrc()
    },
    startOffsetTime () {
      return this.video.startOffsetTime
    },
    textTracks () {
      return this.video.textTracks
    },
    videoTracks () {
      return this.video.videoTracks
    },
    audioTracks () {
      return this.video.audioTracks
    },
    seekable () {
      return this.video.seekable
    },
    buffered () {
      return this.video.buffered
    },
    readyState () {
      return this.video.readyState
    },
    networkState () {
      return this.video.networkState
    }
  },
  created () {
    this.$lockingParams = {
      currentTime: 0,
      playbackRate: 0,
      volume: 0
    }
    if (this.autoplay) {
      // console.warn('自动播放在某系浏览器下不可用')
    }
  },
  mounted () {
    const ext = this.getExtension()
    if (ext) {
      this.callExtensionHook(ext, 'create', this.src)
    } else {
      this.videoSrc = this.src
    }
  },
  methods: {
    setOrUpdateParam (paramName, propName) {
      propName = propName || paramName
      if (typeof this[paramName] === 'number') {
        this.video[paramName] = this[paramName]
      } else {
        this.$emit(`update:${propName}`, this.video[paramName])
      }
    },
    emitUpdateEventWithLock (paramName) {
      ++this.$lockingParams[paramName]
      this.$emit(`update:${paramName}`, this.video[paramName])
      this.$nextTick(() => {
        --this.$lockingParams[paramName]
      })
    },
    updateParamIfNotLocked (paramName, value) {
      if (this.$lockingParams[paramName]) {
        return
      }
      this.video[paramName] = value
    },
    getExtension () {
      if (this.type) {
        return findExtension(this.type)
      } else {
        if (this.src) {
          const ext = /\.([^.]+)(?:[#?]|$)/.exec(this.src)
          if (ext) {
            return findExtension(ext[1])
          }
        }
      }
    },
    callExtensionHook (ext, hook, src) {
      if (typeof ext[hook] === 'function') ext[hook](this, src)
    },
    onAbort () {
      this.$emit('abort')
      this.loading = false
      this.loadingMetaData = false
    },
    onCanPlay () {
      this.canPlay = true
      this.$emit('can-play')
    },
    onCanPlayThrough () {
      this.canPlayThrough = true
      this.$emit('can-play-through')
    },
    onDurationChange () {
      this.duration = this.video.duration
    },
    onEmptied () {
      // console.log('emptied')
    },
    onEnded () {
      this.$emit('ended')
    },
    onError (event) {
      this.$emit('error', event)
    },
    onLoadedData () {
      this.setOrUpdateParam('volume')
      this.setOrUpdateParam('currentTime')
      this.setOrUpdateParam('playbackRate')
      this.setOrUpdateParam('muted')
      this.loading = false
    },
    onLoadedMetaData () {
      this.loadingMetaData = false
    },
    onLoadStart () {
      this.loading = true
      this.loadingMetaData = true
    },
    onPause () {
      this.$emit('update:pause', true)
    },
    onPlay () {
      this.$emit('update:pause', false)
    },
    onPlaying () {
      if (!this.playing) {
        this.playing = true
        this.$emit('playing', true)
      }
    },
    onProgress () {
      // console.log('progress')
    },
    onRateChange () {
      this.emitUpdateEventWithLock('playbackRate')
    },
    onSeeked () {
      this.seeking = false
      this.$emit('seeked')
    },
    onSeeking () {
      this.seeking = true
      this.$emit('seeking')
    },
    onStalled () {
      // console.log('stalled')
    },
    onSuspend () {
      // console.log('suspend')
    },
    onTimeUpdate () {
      this.emitUpdateEventWithLock('currentTime')
    },
    onVolumeChange () {
      this.emitUpdateEventWithLock('volume')
      if (this.muted !== this.video.muted) {
        this.$emit('update:muted', this.video.muted)
      }
    },
    onWaiting () {
      this.playing = false
      this.canPlay = false
      this.$emit('waiting')
    }
  },
  watch: {
    volume (nv) {
      this.updateParamIfNotLocked('volume', nv)
    },
    currentTime (nv) {
      this.updateParamIfNotLocked('currentTime', nv)
    },
    playbackRate (nv) {
      this.updateParamIfNotLocked('playbackRate', nv)
    },
    pause (nv) {
      if (nv) {
        this.video.pause()
      } else {
        this.video.play()
      }
    },
    src (nv) {
      const ext = this.getExtension()
      if (ext) {
        this.callExtensionHook(ext, 'update', nv)
      } else {
        this.videoSrc = nv
      }
    }
  },
  beforeDestroy () {
    const ext = this.getExtension()
    if (ext) {
      this.callExtensionHook(ext, 'destroy')
    }
  },
  $use (ext, {create, update, destroy}) {
    extensions.push({type: ext, create, update, destroy})
  }
}

</script>

<style lang="less">
  .vue-video {
    position: relative;
    video {
      display: block;
      width: 100%;
      height: 100%;
    }
    &-mask {
      position: absolute;
      width: 100%;
      height: 100%;
      left: 0;
      top: 0;
    }
    &-controls {
      position: absolute;
      width: 100%;
      height: 40px;
      left: 0;
      bottom: 0;
    }
  }
</style>
