<template>
  <div id="app">
    <div id="maincontent" v-if="!isEmbed">
      <div id="copyContainer"></div>
      <!--<notification-banner id="1"><a href="https://addons.wago.io">Announcing Wago Addons, the nextgen WoW Addon Platform built by the Wago team. Developer signups now open - click here to get started.</a></notification-banner>-->
      <md-toolbar id="topbar">
        <md-button class="md-icon-button md-hide-small-and-up" @click="toggleMobileNav()">
          <md-icon>menu</md-icon>
        </md-button>
        <h2 class="md-title" @click="$store.commit('setSearchText', '')" id="logo"><router-link to="/"><img src="./assets/wagoio-logo.png"/></router-link></h2>
        <h2 class="md-title" id="xmaslogo" v-if="today.getMonth() === 11"><router-link to="/"><img src="./assets/xmas-hat.png"/></router-link></h2>

        <div id="h-nav" class="md-hide-xsmall">
          <div id="top-search-bar">
            <search-bar ref="searchField" :domain="domain"></search-bar>
          </div>
          <md-input-container id="top-search-domain">
            <md-select v-model="domain">
              <md-option value="0">World of Warcraft</md-option>
              <md-option value="1">Final Fantasy XIV</md-option>
                  </md-select>
                </md-input-container>
        </div>
        <div id="hr-nav" class="md-hide-xsmall">
          <div id="wago-addons-btn"><a href="https://addons.wago.io">Wago Addons</a></div>
          <h2 class="md-title md-hide-small-and-up" id="logo"><router-link to="/"><img src="./assets/wagoio-logo.png"/></router-link></h2>
          <img src="./assets/random.png" id="randombtn" @click="$router.push('/random')" />
          <select-locale display="code"></select-locale>
          <login-button></login-button>
        </div>
      </md-toolbar>
      <md-sidenav class="md-hide-small-and-up md-left" ref="mobileSidebar" id="mobile-sidebar">
          <md-input-container>
          <label>{{ $t('Search game') }}</label>
          <md-select v-model="domain">
            <md-option value="0">World of Warcraft</md-option>
            <md-option value="1">Final Fantasy XIV</md-option>
          </md-select>
          </md-input-container>
        <search-bar ref="searchField" :domain="domain"></search-bar>
        <md-list>
          <md-list-item v-if="LoggedIn">
            <span :class="User.css"><md-icon>person</md-icon> {{ User.name }}</span>
            <md-list-expand>
              <md-list>
                <md-list-item><router-link :to="'/p/'+User.name">{{ $t("My Profile") }}</router-link></md-list-item>
                <md-list-item><router-link to="/my/mentions">{{ $t("My Mentions") }} <span class="unreadCount" v-if="User.unreadMentions.length">{{ User.unreadMentions.length }}</span></router-link></md-list-item>
                <md-list-item><router-link to="/my/stars">{{ $t("My Favorites") }}</router-link></md-list-item>
                <md-list-item><router-link to="/account">{{ $t("Account Settings") }}</router-link></md-list-item>
                <md-list-item v-if="User.access.admin"><router-link to="/admin">Admin</router-link></md-list-item>
              </md-list>
            </md-list-expand>
          </md-list-item>
          <md-list-item v-else><router-link to='/login'>{{ $t("Login") }}</router-link></md-list-item>
          <md-list-item><router-link to='/'>{{ $t("Import") }}</router-link></md-list-item>
          <md-list-item><router-link to='/news'>{{ $t("Site News") }}</router-link></md-list-item>
          <md-list-item><a href="https://addons.wago.io">Wago Addons</a><md-divider></md-divider></md-list-item>
          <md-list-item class="menu-section">World of Warcraft</md-list-item>
          <md-list-item><router-link to='/elvui'>ElvUI</router-link></md-list-item>
          <!--<md-list-item v-if="User && User.access && User.access.beta"><router-link to='/create-new-note'>Encounter Notes [Beta]</router-link></md-list-item>-->
          <!-- <md-list-item><router-link to='/grid2'>Grid2</router-link></md-list-item> -->
          <!--<md-list-item><router-link to='/mdt'>Mythic Dungeon Tools</router-link></md-list-item>-->
          <md-list-item><router-link to='/plater'>Plater Nameplates</router-link></md-list-item>
          <md-list-item><router-link to='/totalrp'>Total RP</router-link></md-list-item>
          <md-list-item><router-link to='/vuhdo'>VuhDo</router-link></md-list-item>
          <md-list-item><router-link to='/weakauras'>WeakAuras</router-link><md-divider></md-divider></md-list-item>
          <md-list-item><router-link to='/classic-weakauras'>Classic WeakAuras</router-link><md-divider></md-divider></md-list-item>
          <md-list-item><router-link to='/tbc-weakauras'>TBC WeakAuras</router-link><md-divider></md-divider></md-list-item>
          <md-list-item class="menu-section">Final Fantasy XIV</md-list-item>
          <md-list-item><router-link to='/delvui'>DelvUI</router-link><md-divider></md-divider></md-list-item>
          <md-list-item class="menu-section">General</md-list-item>
          <md-list-item><router-link to='/collections'>{{ $t("Collections") }}</router-link></md-list-item>
          <md-list-item><router-link to='/snippets'><span class="menu-action" @click="$store.commit('setSearchText', `type:SNIPPET`)">{{ $t("Snippets") }}</span></router-link><md-divider></md-divider></md-list-item>
        </md-list>
      </md-sidenav>
      <md-sidenav class="md-hide-xsmall" ref="full-sidebar" id="full-sidebar">
        <md-list class="mainnav">
          <md-list-item><router-link to='/'>{{ $t("Import") }}</router-link></md-list-item>
          <md-list-item><router-link to='/news'>{{ $t("Site News") }}</router-link><md-divider></md-divider></md-list-item>

<!--
          <md-list-item>
            <div class="md-list-text-container">
              <span class="submenu-single-line">Popular Imports</span>
            </div>
          </md-list-item>
          <md-list-item>
            <router-link to='/weakauras'>
              <div class="md-list-text-container">
                <span>WeakAuras</span>
                <span class="game-select">
                  <router-link to='/weakauras'>Shadowlands</router-link> -
                  <router-link to='/classic-weakauras'>Classic</router-link> -
                  <router-link to='/tbc-weakauras'>TBC</router-link>
                </span>
              </div>
            </router-link>
          </md-list-item>
          <md-list-item>
            <router-link to='/elvui'>
              <div class="md-list-text-container">
                <span class="submenu-single-line">ElvUI</span>
              </div>
            </router-link>
          </md-list-item>
          <md-list-item>
            <router-link to='/plater'>
              <div class="md-list-text-container">
                <span class="submenu-single-line">Plater Nameplates</span>
              </div>
            </router-link>
          </md-list-item>
          <md-list-item>
            <router-link to='/vuhdo'>
              <div class="md-list-text-container">
                <span class="submenu-single-line">VuhDo</span>
              </div>
            </router-link>
          </md-list-item>
          <md-list-item>
            <router-link to='/addons'>
              <div class="md-list-text-container">
                <span class="submenu-single-line">More...</span>
              </div>
            </router-link>
            <md-divider></md-divider>
          </md-list-item>
-->


          <md-list-item class="menu-section">World of Warcraft</md-list-item>
          <md-list-item><router-link to='/elvui'>ElvUI</router-link></md-list-item>
          <!--<md-list-item v-if="User && User.access && User.access.beta"><router-link to='/create-new-note'>Encounter Notes [Beta]</router-link></md-list-item>-->
          <!-- <md-list-item><router-link to='/grid2'>Grid2</router-link></md-list-item> -->
          <!--<md-list-item><router-link to='/mdt'>Mythic Dungeon Tools</router-link></md-list-item>-->
          <md-list-item><router-link to='/opie'>OPie</router-link></md-list-item>
          <md-list-item><router-link to='/plater'>Plater Nameplates</router-link></md-list-item>
          <md-list-item><router-link to='/totalrp'>Total RP</router-link></md-list-item>
          <md-list-item><router-link to='/vuhdo'>VuhDo</router-link></md-list-item>
          <md-list-item class="multi-line">
            <router-link to='/weakauras'>
              <div class="md-list-text-container">
                WeakAuras
                <span class="game-select">
                  <router-link to='/weakauras'>Shadowlands</router-link> - 
                  <router-link to='/classic-weakauras'>Classic</router-link> - 
                  <router-link to='/tbc-weakauras'>TBC</router-link>
                </span>
              </div>
            </router-link>
          </md-list-item>
          <md-list-item class="menu-section">Final Fantasy XIV</md-list-item>
          <md-list-item><router-link to='/delvui'>DelvUI</router-link></md-list-item>
          <md-list-item class="menu-section">General</md-list-item>
          <md-list-item><router-link to='/collections'>{{ $t("Collections") }}</router-link></md-list-item>
          <md-list-item><router-link to='/snippets'><span class="menu-action" @click="$store.commit('setSearchText', `type:SNIPPET`)">{{ $t("Snippets") }}</span></router-link></md-divider></md-list-item>
          <template v-if="LoggedIn">
            <md-list-item class="menu-section">{{ $t("My Data") }}</md-list-item>
            <md-list-item><router-link :to="'/p/'+User.name">{{ $t("My Profile") }}</router-link></md-list-item>
            <md-list-item><router-link to="/my/mentions">{{ $t("My Mentions") }} <span class="unreadCount" v-if="User.unreadMentions.length">{{ User.unreadMentions.length }}</span></router-link></md-list-item>
            <md-list-item><router-link to="/my/stars">{{ $t("My Favorites") }}</router-link><md-divider></md-divider></md-list-item>
          </template>
        </md-list>
        <div v-if="WotM.name" id="wotm">
          <strong>{{ $t("Wago of the Minute") }}</strong><br>
          <router-link :to="'/' + WotM.slug"><strong>{{ WotM.name }}</strong></router-link><br>
          <router-link :to="'/' + WotM.slug"><md-image :md-src="WotM.screenshot" /></router-link>
        </div>

        <ul class="md-list mainnav bottom md-theme-default">
          <li class="md-list-item">
            <a href="https://www.patreon.com/wago" target="_blank" rel="noopener" class="md-list-item-container md-button">
              <div><img src="./assets/Patreon_White.png"></div>{{ $t("Support the website") }}
            </a>
          </li>
          <li class="md-list-item">
            <a href="https://discord.gg/weakauras" target="_blank" rel="noopener" class="md-list-item-container md-button">
              <div><img src="./assets/discord.png"></div>{{ $t("Join WA Discord") }}
            </a>
          </li>
          <li class="md-list-item">
            <a href="https://github.com/oratory/wago.io" target="_blank" rel="noopener" class="md-list-item-container md-button">
              <div><img src="./assets/github.png"></div>{{ $t("Open source") }}
            </a>
          </li>
          <li class="md-list-item">
            <a href="https://weakauras.wtf/" target="_blank" rel="noopener" class="md-list-item-container md-button">
              <div><img src="./assets/weakauralogo.png"></div>WeakAuras Companion
            </a>
          </li>
        </ul>
        <div class='legal'>
          <span>&copy; 2016-{{(new Date()).getFullYear()}} Wago.io</span>
          <span><router-link to="/terms-of-service">{{ $t("Terms of Service") }}</router-link></span>
          <span>
            <router-link to="/privacy-policy">{{ $t("Privacy Policy") }}</router-link>
            <div id="ncmp-consent-link"></div>
          </span>
          <span><a href="mailto:contact@wago.io">{{ $t("Contact") }}</a></span>
          <span data-ccpa-link="1" id="ccpa-content-link"></span>
        </div>
      </md-sidenav>

      <md-snackbar md-position="top center" ref="snackbar" md-duration="4000">
        <span>{{ PopMsg }}</span>
        <md-button @click.native="$refs.snackbar.close()">{{ $t("Close") }}</md-button>
      </md-snackbar>

      <div id="content-frame">
        <md-layout md-row id="content">
          <router-view></router-view>
          <advert ad="wago300x250" :format="$screenWidth > 1025 ? 'video' : ''" fixed="sticky" v-if="this.$store.state.user.UID || this.$store.state.user.guest" />
          <advert ad="wago320x50" :forMobile="true" v-if="this.$store.state.user.UID || this.$store.state.user.guest" />
        </md-layout>
      </div>
    </div>
    <div v-else-if="!$isMobile">
      <view-embed :wagoID="embedID"></view-embed>
    </div>
  </div>
</template>

<script>
function interceptClickEvent (e, vue) {
  var target = e.target || e.srcElement
  // if clicked element is <a class='vr'> then use vue-router
  if (target.tagName === 'A' && target.className.match(/\bvr\b/)) {
    var href = target.getAttribute('href')
    vue.$router.push(href)
    // tell the browser not to respond to the link click
    e.preventDefault()
  }
}

import Advert from './components/UI/Advert.vue'
import SelectLocale from './components/UI/SelectLocale.vue'
import SearchBar from './components/UI/SearchBar.vue'
import LoginButton from './components/UI/LoginButton.vue'
import NotificationBanner from './components/UI/NotificationBanner.vue'
import ViewEmbed from './components/core/ViewEmbed.vue'

export default {
  name: 'app',
  components: {
    'select-locale': SelectLocale,
    'search-bar': SearchBar,
    'login-button': LoginButton,
    'view-embed': ViewEmbed,
    'notification-banner': NotificationBanner,
    'advert': Advert
  },
  data: () => {
    return {
      PopMsg: 'test',
      gSearch: '',
      searchString: '',
      advSearchType: '',
      advSearchUser: '',
      advSearchUserName: '',
      advSearchText: '',
      advSearchStarred: false,
      advSearchMentioned: false,
      advSearchDate: '',
      today: new Date(),
      showAddonsButton: window.localStorage.getItem('notification-1'),
      videoEmbedHTML: '',
    }
  },
  created: function () {
    // listen for link click events at the document level
    if (document.addEventListener) {
      document.addEventListener('click', (e) => {
        if (!e.button) { // leftclick = 0; otherwise firefox overwrites other click actions
          interceptClickEvent(e, this)
        }
      })
    }
    else if (document.attachEvent) {
      document.attachEvent('onclick', (e) => {
        interceptClickEvent(e, this)
      })
    }

    var vue = this
    var params = {}

    if (!window.readCookie('locale')) {
      params.getLocale = 1
    }

    // get user account and locale settings
    if (!this.isEmbed) {
      this.http.get('/account/whoami', params).then((res) => {
        if (res.locale && vue.$store.state.locale !== res.locale) {
          vue.$store.commit('setLocale', res.locale)
        }

        if (res.token) {
          window.setCookie('token', res.token, 365)
          vue.axios.defaults.headers = { 'x-auth-token': res.token }
        }

        // if beta server and user does not have beta access
        // if (process.env.WEB_SERVER.match(/t1000/) && (res.guest || !res.user || !res.user.access || !res.user.access.beta)) {
        //   window.requireBetaAccess = true
        //   if (!vue.$route.path.match(/\/auth\//)) {
        //     window.initPage = vue.$route.path
        //     vue.$router.replace('/login')
        //   }
        // }
        if (res.user) {
          this.$store.commit('setUser', res.user)
          if (vue.$route.path === '/login') {
            vue.$router.replace('/account')
          }
        }
        else if (res.guest) {
          this.$store.commit('setUser', { guest: true, config: { searchOptions: { sort: 'bestmatch', relevance: 'standard', expansion: '' } } })
        }
      }).catch((err) => {
        console.error(err)
        window.initPage = vue.$route.path
        // vue.$router.replace('/login')
      })
    }

    window.eventHub.$on('showSnackBar', this.showSnackBar)

    if (this.$store.getters.i18nLanguage.match(/fa/)) {
      document.body.classList.add('rtl-lang')
    }
  },
  methods: {
    showSnackBar: function (text) {
      this.PopMsg = text
      this.$refs.snackbar.open()
    },
    doLogout: function () {
      this.$router.replace('/logout')
    },
    toggleMobileNav: function () {
      this.$refs.mobileSidebar.toggle()
    },
    goSearch: function (q) {
      this.$refs.advancedSearch.close()
      this.$router.push('/search/' + q.trim().replace(/\s+/g, '+'))
      this.searchString = q.trim()
      this.buildSearch('reset')
    },
    buildSearch: function (root) {
      if (root === 'reset') {
        this.advSearchUser = ''
        this.advSearchUserName = ''
        this.advSearchType = ''
        this.advSearchText = ''
        this.advSearchStarred = false
        this.advSearchMentioned = false
        this.advSearchDate = ''

        this.gSearch = ''
      }
      else if (root) {
        var s = this.gSearch
        var m = s.match(/\btype:\s*"?([a-zA-Z0-9]+)"?\s*/i)
        if (m) {
          this.advSearchType = m[1]
        }

        m = s.match(/\banon:\s*"?([a-zA-Z0-9]+)"?\s*/i)
        if (m && (m[1] === '1' || m[1].toLowerCase() === 'true')) {
          this.advSearchUser = 'anon'
        }

        m = s.match(/\buser:\s*"?([a-zA-Z0-9]+)"?\s*/i)
        if (m) {
          this.advSearchUser = 'user'
          this.advSearchUserName = m[1]
        }

        m = s.match(/\bstarred:\s*"?([a-zA-Z0-9]+)"?\s*/i)
        if (m && (m[1] === '1' || m[1].toLowerCase() === 'true')) {
          this.advSearchStarred = true
        }

        m = s.match(/\bmentioned:\s*"?([a-zA-Z0-9]+)"?\s*/i)
        if (m && (m[1] === '1' || m[1].toLowerCase() === 'true')) {
          this.advSearchMentioned = true
        }

        m = s.match(/\bmodified:\s*"?(\d+\s*[a-zA-Z]+)"?\s*/i)
        if (m) {
          this.advSearchDate = m[1]
        }

        s = s.replace(/\btype:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        s = s.replace(/\banon:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        s = s.replace(/\buser:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        s = s.replace(/\bstarred:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        s = s.replace(/\bmentioned:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        s = s.replace(/\bmodified:\s*"?\d+\s*[a-zA-Z]+"?\s*/i, '')
        this.advSearchText = s.trim()
      }
      else {
        var filters = []
        if (this.advSearchType) {
          filters.push('Type: ' + this.advSearchType)
        }
        this.gSearch = this.gSearch.replace(/\btype:\s*"?[a-zA-Z0-9]+"?\s*/i, '')

        if (this.advSearchUser === 'anon') {
          filters.push('Anon: True')
        }
        else if (this.advSearchUser === 'user' && this.advSearchUserName) {
          filters.push('User: ' + this.advSearchUserName)
        }
        this.gSearch = this.gSearch.replace(/\banon:\s*"?[a-zA-Z0-9]+"?\s*/i, '')
        this.gSearch = this.gSearch.replace(/\buser:\s*"?[a-zA-Z0-9]+"?\s*/i, '')

        if (this.advSearchStarred) {
          filters.push('Starred: True')
        }

        if (this.advSearchMentioned) {
          filters.push('Mentioned: True')
        }

        if (this.advSearchDate) {
          filters.push('Modified: ' + this.advSearchDate)
        }

        if (this.advSearchText) {
          filters.push(this.advSearchText.trim())
        }

        this.gSearch = filters.join(' ')
      }
    },
    autoCompleteUserName: function (q) {
      return this.http.get('/search/username', {name: q.q})
    }
  },
  computed: {
    lang () {
      return this.$store.getters.language
    },
    User () {
      return this.$store.state.user
    },
    LoggedIn () {
      return this.$store.state.loggedIn
    },
    WotM () {
      return this.$store.state.wotm
    },
    Page () {
      return this.$store.state.pageInfo
    },
    isEmbed () {
      return document.getElementById('embed-body')
    },
    embedID () {
      const params = new URLSearchParams(window.location.search)
      return params.get('id')
    },
    domain: {
      get: function () {
        return this.$store.state.domain || '0'
      },
      set: function (v) {
        this.$store.commit('setDomain', v)
      }
    }
  },
  watch: {
    $route (to, from) {
      if (this.$refs.mobileSidebar) {
        this.$refs.mobileSidebar.close()
      }
    },
    advSearchType () {
      this.buildSearch(false)
    },
    advSearchUser (v) {
      if (v === 'user') {
        this.advSearchUserName = ''
        this.$nextTick(() => {
          document.querySelector('#advSearchUserName input').focus()
          document.querySelector('#advSearchUserName').classList.add('md-input-focused')
        })
      }
      this.buildSearch(false)
    },
    advSearchUserName () {
      this.buildSearch(false)
    },
    advSearchText () {
      this.buildSearch(false)
    },
    advSearchStarred () {
      this.buildSearch(false)
    },
    advSearchMentioned () {
      this.buildSearch(false)
    },
    advSearchDate () {
      this.buildSearch(false)
    }
  },
  metaInfo () {
    return {
      titleTemplate: '%s | Wago.io',
      title: this.Page.title,
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1, shrink-to-fit=no' },
        { 'http-equiv': 'x-ua-compatible', content: 'ie=edge' },
        { 'theme-color': 'x-ua-compatible', content: '#c1272d' },
        { name: 'description', content: this.Page.description },

        { name: 'twitter:card', value: 'summary' },

        { name: 'og:title', content: this.Page.title + ' | Wago.io', vmid: 'head-og-title' },
        { name: 'og:type', content: this.$route.path === '/' && 'website' || 'article', vmid: 'head-og-type' },
        { name: 'og:url', content: 'https://wago.io' + this.$route.path, vmid: 'head-og-url' },
        { name: 'og:image', content: this.Page.image, vmid: 'head-og-img' },
        { name: 'og:description', content: this.Page.description, vmid: 'head-og-desc' },

        { name: 'msapplication-square70x70logo', content: 'https://media.wago.io/favicon/smalltile.png' },
        { name: 'msapplication-square150x150logo', content: 'https://media.wago.io/favicon/mediumtile.png' },
        { name: 'msapplication-wide310x150logo', content: 'https://media.wago.io/favicon/widetile.png' },
        { name: 'msapplication-square310x310logo', content: 'https://media.wago.io/favicon/largetile.png' },

        { name: 'robots', content: this.Page.robots }
      ],
      link: [
        { rel: 'stylesheet', href: '//media.wago.io/fonts/fonts.1601783774.css' },

        { rel: 'shortcut icon', href: 'https://media.wago.io/favicon/favicon.ico', type: 'image/x-icon' },
        { rel: 'apple-touch-icon', sizes: '57x57', href: 'https://media.wago.io/favicon/apple-touch-icon-57x57.png' },
        { rel: 'apple-touch-icon', sizes: '60x60', href: 'https://media.wago.io/favicon/apple-touch-icon-60x60.png' },
        { rel: 'apple-touch-icon', sizes: '72x72', href: 'https://media.wago.io/favicon/apple-touch-icon-72x72.png' },
        { rel: 'apple-touch-icon', sizes: '76x76', href: 'https://media.wago.io/favicon/apple-touch-icon-76x76.png' },
        { rel: 'apple-touch-icon', sizes: '114x114', href: 'https://media.wago.io/favicon/apple-touch-icon-114x114.png' },
        { rel: 'apple-touch-icon', sizes: '120x120', href: 'https://media.wago.io/favicon/apple-touch-icon-120x120.png' },
        { rel: 'apple-touch-icon', sizes: '144x144', href: 'https://media.wago.io/favicon/apple-touch-icon-144x144.png' },
        { rel: 'apple-touch-icon', sizes: '152x152', href: 'https://media.wago.io/favicon/apple-touch-icon-152x152.png' },
        { rel: 'apple-touch-icon', sizes: '180x180', href: 'https://media.wago.io/favicon/apple-touch-icon-180x180.png' },
        { rel: 'icon', sizes: '16x16', type: 'image/png', href: 'https://media.wago.io/favicon/favicon-16x16.png' },
        { rel: 'icon', sizes: '32x32', type: 'image/png', href: 'https://media.wago.io/favicon/favicon-32x32.png' },
        { rel: 'icon', sizes: '96x96', type: 'image/png', href: 'https://media.wago.io/favicon/favicon-96x96.png' },
        { rel: 'icon', sizes: '192x192', type: 'image/png', href: 'https://media.wago.io/favicon/android-chrome-192x192.png' }
      ],
      noscript: [
        { innerHTML: 'This website requires JavaScript.' }
      ]
    }
  }
}
</script>

<style lang="scss" src="./assets/themes.scss"></style>
<style lang="scss">
@import './assets/global.css';
body, html {
  height: 100%;
  width: 100%;
  margin: 0;
}
#app {
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  overflow: auto;
}
@media (min-width: 601px) {
  #app { pointer-events: none; }
  #app > * { width: 100%; max-width:100%; height: 100%; overflow: hidden;}
  .md-backdrop { pointer-events: none }
  #topbar {z-index: 9}
  #maincontent {background:linear-gradient(90deg, #333333 0px, #333333 260px, #212121 261px)}

  #full-sidebar .md-sidenav-content {
    top:64px;
    pointer-events: auto;
    transform: translateZ(0)!important;
  }
  #full-sidebar .md-sidenav-content {
    width: 260px;
    display: flex;
    flex-flow: column;
    background: #ECECEC;
    z-index: 1
  }
  #content { padding-left: 260px; padding-bottom: 100px; pointer-events: auto; position: relative; flex-wrap: nowrap;}
  #content > div { flex: 1; width: 100%; }
  #content > div + div { flex: 1; width: inherit; }
  #logo { text-align: left; padding: 8px 16px; }
  #logo img { max-height: 40px; }
  #xmaslogo img { width: 45px; position: absolute; left: 41px; top: -3px;}
  #h-nav { flex: 1 }
  #h-nav, #hr-nav { display: flex }
}

#mobile-anchor {
  position: fixed;
  bottom: 0;
  width: 340px;
  padding: 0;
  height: 64px;
  left: 50%;
  margin: 0 0 0 -170px;
}
#mobile-anchor + div {
  padding-top: 76px;
}
#mobile-sidebar .md-input-container {
  margin: 0 8px 4px;
  width: auto;
}
#mobile-sidebar .search-container {
  min-width: 100%;
  margin: 0;
}

#app {
  min-height: 100%;
  display: flex;
  flex-flow: column nowrap;
  flex: 1;
  pointer-events: auto;
}

#full-sidebar .mainnav { padding: 0; background-color: #ccc; position: relative; display: block}
#full-sidebar .md-sidenav-content { overflow: inherit; bottom: auto }
.mainnav .md-subheader { padding-left: 0;}
.mainnav .md-list-item img { max-height: 32px }
.mainnav .md-list-item a { justify-content: start }
.mainnav .md-list-item a span.game-select, .mainnav .md-list-item a span.unreadCount { margin-left: 8px; line-height: 18px }
.mainnav .md-list-item a span.menu-action { width: 100% }
.mainnav .md-list-item { height: 36px }
.mainnav .md-list-item.multi-line { height: 40px }
.mainnav .md-list-item .md-list-item-container { min-height: auto; }
.mainnav .md-list-item.small { height: 16px; }
.mainnav .md-list-item.small .md-list-item-container { font-size: 12px; height: 18px; }
.mainnav.bottom .md-list-item-container { font-size: 90%; padding-right: 0 }
.mainnav.bottom .md-list-item-container img { max-width: 74px; }
.mainnav.bottom .md-list-item a div { display: inline-block; width: 90px;  }

#wago-addons-btn {border: 1px solid #a12126; margin: 4px 16px; line-height: 30px; color: white; background: #2a090a url('./assets/robot.png'); background-size: 27%; background-repeat: no-repeat; background-position: top right;}
#wago-addons-btn a {color: inherit; display: block; padding: 4px 52px 4px 12px; }
#wago-addons-btn:hover {background-color: #1d0607}
#wago-addons-btn a:hover {text-decoration: none;}

#side-bottom {}
#side-bottom .resource {  display: inline-block; padding: 8px 0 0 16px; color: default }
#side-bottom .resource img { max-width: 30%; }
.md-sidenav h2 { background: black; margin:0; padding: 16px 24px; text-align: center}
#user-info { background: #CCC; padding: 16px 16px 0 }
#gSearch { margin-left: 16px; white-space: nowrap; flex: 1; max-width: 640px; }
#gSearch input { border:0; padding: 0 16px; line-height:40px; max-width: 640px;  width: 100%; outline: none; }
#gSearch button { margin-left: -48px }
#gSearch .md-menu { display: inline }
#advancedSearch { padding: 16px; width: 640px; max-width: 640px; top:64px!important; left: 120px!important; box-shadow: 0 7px 9px -4px rgba(0, 0, 0, 0.2), 0 14px 21px 2px rgba(0, 0, 0, 0.14), 0 5px 26px 4px rgba(0, 0, 0, 0.12) }
#advancedSearch .md-list:after { content:none }
#advancedSearch .md-checkbox { margin-right: 32px; font-size: 12px; font-weight: normal; }
.advSearchSelect.md-select-content { max-height: initial; min-width: 220px; margin-left: 0; margin-top: 36px!important }
.advSearchButtons { flex-direction: row-reverse; }

.md-list { padding-bottom: 0}
.md-list:after { height: 1px; width:100%; background-color: rgba(0,0,0,.12); content: " " }
.md-list-item .md-list-item-container { padding-left: 16px }
#wotm { padding: 16px }
#wotm img { max-height: 200px }
#wotm a { max-width: 100%; overflow: hidden; text-overflow: ellipsis; display: inline-block; }
.wotm-controls { padding: 16px 16px 0; margin-bottom: -8px}
.wotm-controls button { background: none; border: none; cursor: pointer}
.legal { padding: 16px; }
.legal > span { font-size: 90%; padding: 0 0 8px; display: block; }
#ncmp-consent-link > button { font-size: inherit; padding: 10px 0 2px; display: block; border: 0; background: none; color: #d7373d; cursor: pointer }
#ncmp-consent-link > button:hover { color: #ad1457; text-decoration: underline }

@media (max-width: 800px) {
  #gSearch button { display: none }
}

@media (max-width: 600px) {
  .md-toolbar h2 { margin:0; padding:0}
  footer .legal { float: none; padding-bottom: 6px}
  footer .legal span { display: inline; padding-left: 6px }
  footer .resource { padding-left: 0; width:50%}
  #logo img { max-height: 40px }
  #xmaslogo img {width: 43px; position: absolute; left: 81px; top: -2px;}
}

.document { padding: 16px }
.md-card { padding: 16px; margin: 16px; background: white }
.md-card > h3 { margin: 0 }
.md-card > h3 + .md-subheader { padding:0; min-height:0 }

.unreadCount { background: #c1272d; color: white; padding: 4px; border-radius: 2px;  }
.md-snackbar-container { border: 1px solid black }

#randombtn { height: 36px; opacity: 0.7; transition: all 1s ease-in; cursor: pointer; margin: 6px 8px }

body.theme-dark .md-input-container label a { -webkit-text-fill-color: initial }

.ads-enabled #app {padding-bottom: 60px}
@media only screen and (min-width: 1025px) {
  .ads-enabled #content > :first-child {max-width: calc(100% - 425px);}
}

.submenu-single-line {line-height: 36px!important;  min-height: 36px; display: block;}
.game-select a {color: #999!important; font-size: 14px;}
.game-select a:hover {text-decoration: none!important; color: #BBB!important}

.md-list-item.menu-section {
  background: #333333;
  position: relative;
}
.md-list-item.menu-section .md-list-item-container {
  font-weight: bold;
  font-size: 90%;
  min-height: 36px;
  line-height: 36px;
}


#top-search-domain {
  width: auto;
  height: 40px;
  border-radius: 4px;
  margin-left: 4px;
  align-self: center;
  background: #404040;
  padding: 4px 16px;
  &:after {
    background-color: transparent!important;
  }
}
</style>