<template>
  <div class="home">
    <audio controls ref="myAudio">
      <source src="/Jazz_Mango.mp3" type="audio/mpeg">
    Your browser does not support the audio element.
    </audio>
    <VolumeBar></VolumeBar>
    <canvas ref="myCanvas"></canvas>
  </div>
</template>

<script>
// @ is an alias to /src
import VolumeBar from '@/components/VolumeBar.vue'

export default {
  name: 'home',
  data () {
    return {
      audioCtx: null,
      analyser: null,
      ctx: null,
      bufferLength: null,
      dataArray: null
    }
  },
  components: {
    VolumeBar
  },
  methods: {
    draw () {
      const canvas = this.$refs['myCanvas']
      this.animation = window.requestAnimationFrame(this.draw)
      this.analyser.getByteTimeDomainData(this.dataArray)
      this.ctx.fillStyle = 'rgb(200,200,200)'
      this.ctx.fillRect(0, 0, canvas.width, canvas.height)

      this.ctx.lineWidth = 2
      this.ctx.storkeStyle = 'rgb(0,0,0)'
      this.ctx.beginPath()

      let sliceWidth = canvas.width * 1.0 / this.bufferLength
      let x = 0

      for (let i = 0; i < this.bufferLength; i++) {
        let y = canvas.height / 2
        let v = this.dataArray[i] / 128
        if (i === 0) {
          this.ctx.moveTo(x, y)
        } else if (x < canvas.width / 4 || x > canvas.width * 3 / 4) {

        } else {
          this.ctx.lineTo(x, y * v)
        }

        x += sliceWidth
        // let v = this.dataArray[i] / 128.0
        // let y = canvas.height / 2 * v
        // if (i === 0) {
        //   this.ctx.moveTo(x, y)
        // } else {
        //   this.ctx.lineTo(x, y)
        // }
        // x += sliceWidth
      }
      this.ctx.lineTo(canvas.width, canvas.height / 2)
      this.ctx.stroke()
    }
  },
  mounted () {
    const audio = this.$refs['myAudio']
    this.audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    const track = this.audioCtx.createMediaElementSource(audio)
    const gainNode = this.audioCtx.createGain()
    const processor = this.audioCtx.createScriptProcessor(4096, 1, 1)
    this.analyser = this.audioCtx.createAnalyser()
    this.analyser.fftSize = 2048
    this.bufferLength = this.analyser.frequencyBinCount

    this.dataArray = new Uint8Array(this.bufferLength)
    this.analyser.getByteTimeDomainData(this.dataArray)

    const canvas = this.$refs['myCanvas']
    this.ctx = canvas.getContext('2d')
    this.draw()
    track.connect(gainNode).connect(this.analyser).connect(this.audioCtx.destination)
    // processor.onaudioprocess = function (e) {
    //   let input = e.inputBuffer.getChannelData(0)
    //   let output = e.outputBuffer.getChannelData(0)
    //   for (let i = 0; i < input.length; i++) {
    //     output[i] = input[i]
    //   }
    //   this.arr = output
    // }
  },
  beforeDestroy () {
    window.cancelAnimationFrame(this.animation)
  }
}
</script>
