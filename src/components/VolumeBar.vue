<template>
  <div ref="volume-bar" class="volume-bar" @mouseover="changeVol">
    <canvas ref="myCanvas"></canvas>
    <div class="line">
      <img src="@/assets/icon/Voice.svg" class="voice" draggable="false" @mouseup="dragVoiceEnd" @mousedown="dragVoice">
    </div>
  </div>
</template>

<script>
export default {
  name: 'volume-bar',
  data () {
    return {
      ctx: null,
      bufferLength: 2048,
      dataArray: null,
      animation: null
    }
  },
  methods: {
    draw () {
      const canvas = this.$refs['myCanvas']
      this.animation = window.requestAnimationFrame(this.draw)
      this.analyser.getByteTimeDomainData(this.dataArray)
      let grd = this.ctx.createLinearGradient(0, 0, 0, canvas.height)
      grd.addColorStop(0, '#B0DDEB')
      grd.addColorStop(1, '#639CF2')
      this.ctx.fillStyle = grd
      this.ctx.fillRect(0, 0, canvas.width, canvas.height)
      this.ctx.lineWidth = 10
      this.ctx.strokeStyle = 'white'
      let sliceWidth = canvas.width * 1.0 / this.bufferLength * this.controlBar
      let x = 10
      for (let i = 0; i < this.bufferLength / this.controlBar; i++) {
        this.ctx.beginPath()
        let y = canvas.height
        let v = this.dataArray[i * this.controlBar] / 128
        this.ctx.strokeStyle = 'white'
        this.ctx.moveTo(x, y)
        this.ctx.lineTo(x, y - v * v * 20)
        this.ctx.stroke()
        this.ctx.beginPath()
        this.ctx.strokeStyle = 'rgba(255,255,255, 0.3)'
        this.ctx.moveTo(x, y)
        this.ctx.lineTo(x, y - v * v * 60)
        x += sliceWidth
        this.ctx.stroke()
      }
    },
    changeVol (e) {
      if (!this.isDragVoice) return
      console.log(e)
    },
    dragVoice () {
      this.isDragVoice = true
      console.log(123)
    },
    dragVoiceEnd () {
      this.isDragVoice = false
      console.log(456)
    }
  },
  props: {
    value: Number,
    audioCtx: AudioContext,
    isPlay: Boolean,
    analyser: AnalyserNode,
    controlBar: Number
  },
  watch: {
    isPlay (val) {
      if (val) {
        window.requestAnimationFrame(this.draw)
      } else {
        cancelAnimationFrame(this.animation)
      }
    }
  },
  mounted () {
    const canvas = this.$refs['myCanvas']
    canvas.width = this.$refs['volume-bar'].offsetWidth
    this.ctx = canvas.getContext('2d')
    this.bufferLength = this.analyser.frequencyBinCount
    this.dataArray = new Uint8Array(this.bufferLength)
    console.log(this.analyser)
    this.analyser.getByteTimeDomainData(this.dataArray)
  },
  beforeDestroy () {
    cancelAnimationFrame(this.animation)
  }
}
</script>

<style lang="stylus" scoped>
  .volume-bar
    position relative
    background: linear-gradient(#B0DDEB, #639CF2)
    .line
      background linear-gradient(to left, #F6FAFF, #BDDBE4)
      height 3px
      position absolute
      width 100%
      bottom 50%
      box-shadow 0 3px 3px rgba(#000, 0.3)

    canvas
      margin-bottom 10px
      box-shadow: 0px 2px 3px rgba(54, 91, 89, 0.5)
    .voice
      position absolute
      right 10%
      transform translateY(-50%)
      cursor pointer
</style>
