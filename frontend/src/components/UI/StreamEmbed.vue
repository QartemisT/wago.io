<template>
  <div class="embed-container" v-if="stream !== '__streamspread' && visible && showAds">
    <div ref="player" class="embed-player"></div>
    <div class="embed-player-close" @click="closeStream()" v-if="twitchOn">
      <img class="embed-player-close-icon" src="./../../assets/stream-close.png">
    </div>
  </div>
</template>

<script>
export default {
  name: 'embedstream',
  props: { stream: String, preview: Boolean },
  data: () => {
    return {
      screenWidth: window.innerWidth,
      visible: true,
      source: '',
      loadJS: null,
      twitchOn: false,
      twitchPlayer: null,
      loaded: false,
      refresh: null,
      muted: true
    }
  },

  created () {
    window.addEventListener('resize', this.handleResize)
    const _this = this

    this.refresh = setInterval(function () {
      if (_this.visible && _this.twitchPlayer && _this.twitchPlayer.getMuted()) {
        _this.visible = false
        _this.muted = _this.twitchPlayer.getMuted()
        setTimeout(function () {
          _this.visible = true
          _this.loadEmbed()
        }, 1000)
      }
    }, 1000*60*20)
  },
  destroyed () {
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    closeStream () {
      this.visible = false
      // this.http.post('/account/close-embed')
    },

    handleResize () {
      this.screenWidth = window.innerWidth
    },

    async loadEmbed () {
      if (!this.showAds) {
        return false
      }
      if (this.stream === '__streamspread' && this.showAds) {
        this.loadJS = new Promise((resolve) => {
          let body = document.querySelector('body')
          let streamspread = document.createElement('script')
          streamspread.src = 'https://adc.streamspread.com/js/fd23dd90-b346-4a09-ae30-817beb89a23a.js'
          streamspread.async = true
          streamspread.onload = resolve
          body.appendChild(streamspread)
        })
        this.source = 'streamspread'
      }
      else if (this.stream === '__closed') {
        this.visible = false
      }
      else if (!this.twitchPlayer) {
        this.loadJS = new Promise((resolve) => {
          let body = document.querySelector('body')
          let streamspread = document.createElement('script')
          streamspread.src = 'https://player.twitch.tv/js/embed/v1.js'
          streamspread.async = true
          streamspread.onload = resolve
          body.appendChild(streamspread)
        })
        this.source = 'twitch'
      }

      if (this.source === 'twitch') {
        await this.loadJS
        this.twitchPlayer = new window.Twitch.Player(this.$refs.player, {
          width: 426,
          height: 240,
          channel: this.stream,
          muted: this.muted
        });
        this.twitchPlayer.play()
        this.twitchOn = true
      }
    }
  },
  watch: {
    $route (to, from) {
      if (this.adNetwork === 'nitropay') {
        let t = Date.now()
        if (this.debounce + 1000 < t) {
          this.debounce = t
          this.uid++
          this.$nextTick(function () {
            this.insertAds()
          })
        }
      }
    }
  },
  beforeDestroy: function () {
  },
  computed: {
    showAds () {
      if (this.$isMobile) {
        return false
      }
      var isEmbed = document.getElementById('embed-body')
      var user = this.$store.state.user
      if (isEmbed || (!Object.keys(user).length || user.hideAds) || this.screenWidth < 452) {
        return false
      }

      this.$nextTick(() => {
        this.loadEmbed()
      })
      return true
    },
  }
}
</script>

<style>
.embed-container {width: 426px; height: 240px; position: fixed; z-index: 999999; right: 2px; bottom: 2px; margin: auto!important; overflow: hidden; text-align: center!important; border-radius: 2px;}
.embed-player {width: 426px; height: 240px;}
.embed-player iframe, .embed-player iframe:not(.md-image) {height: inherit}
.embed-player.preview {position: relative}
.embed-player-close {width: 26px; height: 26px;cursor: pointer; background-color: #2A2530; border-radius: 50%; text-align: center; display: inline-block; opacity: 0.7; position: absolute; top: 8px; right: 8px}
.embed-player-close:hover {opacity: 1}
.embed-player-close-icon {width: 10px; height: auto; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);}
</style>
