<template>
  <div class="audio" :style="`height: ${audioHeight}px;`">
    <audio
      :src="musicSrc"
      crossorigin="anonymous"
      ref="audio">
    </audio>
    <canvas class="music_canvas" ref="canvas"></canvas>
    <div class="controlls">
      <button @click="playSong" class="controll_btn">
        <i :class="['iconfont', `icon-${isPlay ? 'zanting' : 'bofang'}`]"></i>
      </button>
      <button @click="nextSong" class="controll_btn">
        <i class="iconfont icon-next"></i>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    audioHeight: {
      type: Number,
      default: 300
    },
    musicSrc: {
      type: String,
      require: true
    }
  },
  data () {
    return {
      dataArray: [],
      timer: null,
      analyser: null,
      ctx: null,
      bufferLength: null,
      source: null,
      canvas: null,
      isPlay: false,
      barValue: 0
    }
  },
  methods: {
    playSong () {
      if (!this.bufferLength) {
        this.initAudio()
      } else {
        if (this.isPlay) {
          this.isPlay = false
          this.$refs.audio.pause()
        } else {
          this.isPlay = true
          this.$refs.audio.play()
        }
      }
    },
    nextSong () {},
    initAudio () {
      const ac = new window.AudioContext()
      this.analyser = ac.createAnalyser()
      this.analyser.fftSize = 512
      this.bufferLength = this.analyser.fftSize / 2
      const audio = this.$refs.audio
      this.source = ac.createMediaElementSource(audio)
      this.source.connect(this.analyser)
      this.analyser.connect(ac.destination)
      this.bufferLength = this.analyser.frequencyBinCount
      this.isPlay = true
      audio.play()
      this.initCanvas()
    },
    initCanvas () {
      this.canvas = this.$refs.canvas
      this.canvas.width = this.canvas.clientWidth
      this.canvas.height = this.canvas.clientHeight
      this.ctx = this.canvas.getContext('2d')
      requestAnimationFrame(this.renderCanavs)
    },
    renderCanavs () {
      this.dataArray = new Uint8Array(this.bufferLength)
      this.analyser.getByteFrequencyData(this.dataArray)
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
      const barWidth = this.canvas.width / this.bufferLength * 2.3
      this.dataArray.forEach((item, index) => {
        const barHeight = item
        const r = 50
        const g = 250 * (index / this.bufferLength)
        const b = barHeight + 25 * (index / this.bufferLength)
        this.ctx.fillStyle = `rgba(${r}, ${g}, ${b}, .7)`
        this.ctx.fillRect(barWidth * (index + 1), this.canvas.height - barHeight, barWidth, barHeight)
      })
      requestAnimationFrame(this.renderCanavs)
    }
  },
  beforeDestroy () {
    if (this.timer) {
      clearInterval(this.timer)
    }
    cancelAnimationFrame(this.renderCanavs)
  }
}
</script>

<style lang="less" scoped>
@theme: #2376b7;

.audio {
  position: relative;
  width: 100%;
  .music_canvas {
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 2;
  };
  .controlls {
    display: flex;
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    justify-content: center;
    background: url(http://www.paradx.cn/img/night.jpg);
    background-size: 100% 100%;
    .controll_btn {
      position: relative;
      cursor: pointer;
      margin: 0 10px;
      top: 70%;
      width: 40px;
      height: 40px;
      border-radius: 50%;
      color: @theme;
      border: 2px solid @theme;
      background-color: rgba(0, 0, 0, 0);
      z-index: 2;
      box-shadow: 0 0 15px #2376b7;
      &:active {
        transform: scale(.9)
      };
    }
  };
};
</style>
