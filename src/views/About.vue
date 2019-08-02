<template>
  <div class="home" ref="home">
    <audio controls ref="myAudio">
      <source src="/test.mp3" type="audio/mpeg">
    Your browser does not support the audio element.
    </audio>
    <VolumeBar></VolumeBar>
    <canvas ref="myCanvas"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import VolumeBar from '@/components/VolumeBar.vue'
import { clearTimeout } from 'timers'

export default {
  name: 'about',
  data () {
    return {
      audioCtx: null,
      analyser: null,
      ctx: null,
      bufferLength: null,
      dataArray: null,
      controlBar: 8
    }
  },
  components: {
    VolumeBar
  },
  methods: {
    draw () {
      const canvas = this.$refs['myCanvas']
      this.animation = window.setTimeout(this.draw, 30)
      this.analyser.getByteTimeDomainData(this.dataArray)
      let grd = this.ctx.createLinearGradient(0, 0, 0, canvas.height)
      grd.addColorStop(0, '#B0DDEB')
      grd.addColorStop(1, '#639CF2')
      this.ctx.fillStyle = grd
      this.ctx.fillRect(0, 0, canvas.width, canvas.height)

      this.ctx.lineWidth = 10
      this.ctx.strokeStyle = 'white'
      let sliceWidth = canvas.width * 1.0 / this.bufferLength * this.controlBar
      let x = 0

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
    }
  },
  mounted () {
    const audio = this.$refs['myAudio']
    const canvas = this.$refs['myCanvas']
    canvas.width = this.$refs['home'].offsetWidth
    this.audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    const track = this.audioCtx.createMediaElementSource(audio)
    const gainNode = this.audioCtx.createGain()
    this.analyser = this.audioCtx.createAnalyser()
    this.analyser.fftSize = 2048
    this.bufferLength = this.analyser.frequencyBinCount

    this.dataArray = new Uint8Array(this.bufferLength)
    this.analyser.getByteTimeDomainData(this.dataArray)
    this.ctx = canvas.getContext('2d')
    this.draw()
    track.connect(gainNode).connect(this.analyser).connect(this.audioCtx.destination)
  },
  beforeDestroy () {
    clearTimeout(this.animation)
  }
}
</script>
