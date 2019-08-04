<template>
  <div class="home" ref="home">
    <div class="home__player">
      <div class="player__top">
        <div class="player__cover"
          :style="{
            backgroundImage: `url(${activeSong.pictureUrl})`
          }"
        ></div>
        <div class="player__main">
          <h1>{{ activeSong.title }}</h1>
          <h2>{{ activeSong.author }}</h2>
          <div class="player__actions">
            <button ><img src="@/assets/icon/Repeat.svg" alt="repeat"></button>
            <button class="border-right" @click="toPrevSong"><img src="@/assets/icon/Component-1.svg" alt=""></button>
            <div class="action__center">
              <button class="actions__main" @click="togglePlay">
                <img src="@/assets/icon/play.svg" height="35px" v-if="!isPlay">
                <img src="@/assets/icon/pause.svg" v-else height="27px">
              </button>
            </div>
            <button class="border-left" @click="toNextSong"><img src="@/assets/icon/Component.svg" alt=""></button>
            <button><img src="@/assets/icon/Cycle.svg" alt=""></button>
          </div>
        </div>
      </div>
      <audio controls ref="myAudio" id="audio" @ended="isPlay = false" muted :autoplay="true">
        <source :src="activeSong.url">
      Your browser does not support the audio element.
      </audio>
      <VolumeBar v-if="analyser"
        :isPlay="isPlay"
        :analyser="analyser"
        :controlBar="controlBar"
        :audioCtx="audioCtx"></VolumeBar>
    </div>
    <div class="home__list">
      <div class="list__top">
        <div class="top__actions">
          <button><img src="@/assets/icon/arrow.svg" alt="repeat"></button>
          <button class="small"><img src="@/assets/icon/Home.svg" alt="repeat"></button>
          <button class="small"><img src="@/assets/icon/Search.svg" alt="repeat"></button>
          <button class="small"><img src="@/assets/icon/Stock.svg" alt="repeat"></button>
        </div>
        <div class="top__next">
          <ul class="song__list" >
            <li
              :class="{active: i === songPointer}"
              class="song__detail p-0">
              <div
                class="thumbnail"
                :style="{
                  backgroundImage: `url(${nextSong.pictureUrl})`
                }"
              ></div>
              <div class="detail__info">
                <h1>Next Music</h1>
                <h2>{{ nextSong.title }}</h2>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <div class="list__main">
        <ul class="song__list" >
          <li
            v-for="(song, i) in mediaList" :key="i"
            class="song__detail" :class="{active: i === songPointer}"
            @click="songPointer = i"
          >
            <div class="thumbnail" :style="{
              backgroundImage: `url(${song.pictureUrl})`
            }"></div>
            <div class="detail__info">
              <h1>{{ song.title }}</h1>
              <h2>{{ song.author }}</h2>
            </div>
          </li>
        </ul>
      </div>
    </div>

  </div>
</template>

<script>
// @ is an alias to /src
import VolumeBar from '@/components/VolumeBar.vue'

export default {
  name: 'about',
  data () {
    return {
      audioCtx: null,
      analyser: null,
      ctx: null,
      bufferLength: null,
      dataArray: null,
      controlBar: 8,
      isPlay: false,
      songPointer: 0,
      mediaList: [{
        url: './Ice_Cream.mp3',
        author: 'Joey Pecoraro',
        title: 'Ice Crame',
        pictureUrl: './SongExploder109-FleetFoxes-600x420.png'
      }, {
        url: './Jazz_Mango.mp3',
        author: 'Joey Pecoraro',
        title: 'Jazz Mango',
        pictureUrl: ''
      }, {
        url: './How_it_Began.mp3',
        author: 'Silent Partner',
        title: 'How it Began',
        pictureUrl: ''
      }]
    }
  },
  computed: {
    activeSong () {
      return this.mediaList[this.songPointer]
    },
    nextSong () {
      let next = this.songPointer + 1
      return this.mediaList[next % this.mediaList.length]
    }
  },
  components: {
    VolumeBar
  },
  methods: {
    toNextSong () {
      this.songPointer += 1
      this.songPointer = this.songPointer % this.mediaList.length
    },
    toPrevSong () {
      this.songPointer -= 1
      this.songPointer = this.songPointer < 0 ? this.mediaList.length - 1 : this.songPointer
    },
    togglePlay () {
      const audio = this.$refs['myAudio']
      if (audio.paused) {
        audio.play()
        this.audioCtx.resume().then(() => {
          console.log('Playback resumed successfully')
        })
        this.isPlay = true
      } else {
        audio.pause()
        this.isPlay = false
      }
    },
    async initData () {
      let res = await fetch('https://api.thecatapi.com/v1/images/search?limit=2&page=1')
      let cats = await res.json()
      cats.forEach((cat, i) => {
        this.mediaList[i + 1].pictureUrl = cat.url
      })
    }
  },
  watch: {
    activeSong () {
      const audio = this.$refs.myAudio
      audio.load()

      if (this.isPlay) {
        audio.play()
      } else {
        audio.pause()
      }
    }
  },
  mounted () {
    const audio = this.$refs['myAudio']

    this.audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    const track = this.audioCtx.createMediaElementSource(audio)
    const gainNode = this.audioCtx.createGain()
    this.analyser = this.audioCtx.createAnalyser()
    this.analyser.fftSize = 2048
    track.connect(gainNode).connect(this.analyser).connect(this.audioCtx.destination)
    audio.load()
    audio.pause()
    this.initData()
  },
  beforeDestroy () {
  }
}
</script>

<style lang="stylus">
  @import url('https://fonts.googleapis.com/css?family=Roboto&display=swap')
  audio
    display none
  .home
    min-height 100vh
    display flex
    &__player
      flex 4
      display flex
      flex-direction column
      position relative
      .player__top
        flex 1
        background linear-gradient(135deg, #BDDBE4, #F6FAFF)
        display flex
        align-items flex-end
    &__list
      flex 1
      background linear-gradient(360deg, #BEDCE6 1.34%, #F7FAFF 97.17%)
      box-shadow -2px 0px 10px rgba(0, 0, 0, 0.25)
      z-index 1
  .player__cover
    position absolute
    top -270px
    left -20px
    width 991px
    background-color #fff
    height 991px
    background-image url('../assets/SongExploder109-FleetFoxes-600x420.png')
    background-size 100%
    background-position 0 100%
    background-repeat no-repeat
    border-radius 0 50% 50% 50%
    box-shadow 0px 4px 20px #93DEF5
  .player__main
    min-width 37%
    margin-left auto
    text-align left
    margin-bottom 100px
    h1
      font-family Roboto
      font-style normal
      font-weight bold
      font-size 32px
      line-height 37px
      letter-spacing 0.1em
      color #365B59
    h2
      font-family Roboto
      font-style normal
      font-weight normal
      font-size 20px
      line-height 23px
      letter-spacing 0.1em
      color #717171
.home__list
  display flex
  flex-direction column
  .list__top
    height 180px
    box-shadow 0 5px 5px rgba(black, 0.1)
    z-index 0
    padding  40px 50px 0 50px
    display flex
    flex-direction column
    justify-content space-between
  .list__main
    flex 1
    background linear-gradient(135deg, #F6FAFF, #BDDBE4)
.player__actions
  display flex
  .action__center
    padding 0 40px
  button
    background-color transparent
    border none
    outline none
    cursor pointer
    flex 1
  .actions__main
    display flex
    align-items center
    justify-content center
    border-radius 50%
    width 50px
    height 50px
    border 1px solid #639CF2
  .border-right
    border-right 1px solid #c4c4c4
  .border-left
    border-left 1px solid #c4c4c4
.song__list
  list-style none
  margin 0
  padding 0
  .song__detail
    cursor pointer
    &.active
      background rgba(99, 156, 242, 0.2)
      box-shadow inset 0px 4px 4px rgba(0, 0, 0, 0.15), inset 0px -3px 4px rgba(0, 0, 0, 0.1)
    display flex
    align-items center
    padding 8px 50px
    .thumbnail
      width 70px
      height 70px
      border-radius 50%
      background-image url('../assets/SongExploder109-FleetFoxes-600x420.png')
      background-size cover
      background-position center center
    .detail__info
      text-align left
      margin-left 10px

      h1
        font-family Roboto
        font-style normal
        font-weight 500
        font-size 20px
        line-height 23px
        letter-spacing 0.1em
        color #365B59
      h2
        font-family Roboto
        font-style normal
        font-weight normal
        font-size 18px
        line-height 21px
        letter-spacing 0.1em
        color #717171
.top__actions
  display flex
  button
    background-color transparent
    border none
    outline none
    cursor pointer
    flex 1
    max-width 100px
    text-align left
    &:first-child
      margin-right auto
    &.small
      max-width 50px
.p-0
  padding 0!important
</style>
