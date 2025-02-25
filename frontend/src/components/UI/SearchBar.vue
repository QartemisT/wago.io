<template>
  <div>
    <div class="search-dropdown-block" v-if="dropdownMenu" @click="dropdownMenu=''"></div>
    <div class="search-container" ref="container">
      <div class="search-icon" @click="submitSearch()"><md-icon>search</md-icon></div>
      <div :class="[{'search-input': true, hasTag: inputHasTag}, searchMode+'-search']" :id="searchID" @keydown="checkSearchSubmit" spellcheck="false" ref="searchInput" @click="openDropdownMenu()"></div>
    </div>
    <div :class="{'search-dropdown': true, hidden: dropdownMenu === ''}" ref="searchDropdown">
      <md-list v-if="dropdownMenu === 'main'">
        <md-list-item><h4>{{ $t('Search Options') }}</h4></md-list-item>
        <md-list-item v-if="!domain" @click="addSearchText('Type:')"><strong>{{ $t('Type') }}</strong><span>{{ $t('Example') }}: {{ $t('WeakAura; Plater') }}</span></md-list-item>
        <md-list-item v-else-if="domain === 1" @click="addSearchText('Type:')"><strong>{{ $t('Type') }}</strong><span>{{ $t('Example') }}: {{ $t('DelvUI') }}</span></md-list-item>
        <md-list-item v-if="!domain" @click="addSearchText('Expansion:')"><strong>{{ $t('Expansion') }}</strong><span>{{ $t('Example') }}: {{ $t('Shadowlands; TBCC') }}</span></md-list-item>
        <md-list-item @click="addSearchText('Metric:')"><strong>{{ $t('Metric') }}</strong><span>{{ $t('Example') }}: {{ $t('Installs>150; Stars>90') }}</span></md-list-item>
        <md-list-item @click="addSearchText('Date:')"><strong>{{ $t('Date') }}</strong><span>{{ $t('Example') }}: {{exampleDate}}</span></md-list-item>
        <md-list-item v-if="!domain && (searchMode !== 'standard' || betaUser)"><h4>{{ $t('Search Mode') }}</h4></md-list-item>
        <md-list-item v-if="searchMode !== 'standard'" @click="setSearchMode('standard')"><strong>{{ $t('Standard Search') }}</strong><span>{{ $t('Search imports by text and/or filters') }}</span></md-list-item>
        <md-list-item v-if="!domain && (searchMode !== 'code' && betaUser)" @click="setSearchMode('code')"><strong>[Beta] {{ $t('Code Search') }}</strong><span>{{ $t('Search the custom code of imports') }}</span></md-list-item>
      </md-list>
      <md-list v-else-if="dropdownMenu === 'expansion' && !domain">
        <md-list-item><h4>{{ $t('Expansion Options') }}<span class="close-search" @click="clearSearch(/expansion:\s*/i)">✖</span></h4></md-list-item>
        <md-list-item @click="replaceSearchText(/expansion:\s*/i, 'expansion:sl')">{{ $t('Shadowlands') }}</md-list-item>
        <md-list-item @click="replaceSearchText(/expansion:\s*/i, 'expansion:tbc')">{{ $t('The Burning Crusade Classic') }}</md-list-item>
        <md-list-item @click="replaceSearchText(/expansion:\s*/i, 'expansion:classic')">{{ $t('Classic') }}</md-list-item>
        <md-list-item><em>{{ $t('Expansion filter is only applied to WeakAura imports') }}</em></md-list-item>
      </md-list>
      <md-list v-else-if="dropdownMenu === 'type'">
        <md-list-item><h4>{{ $t('Import Type Options') }}<span class="close-search" @click="clearSearch(/type:\s*/i)">✖</span></h4></md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:WEAKAURA')">WeakAura</md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:PLATER')">Plater</md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:ELVUI')">ElvUI</md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:VUHDO')">VuhDo</md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:OPIE')">OPie</md-list-item>
        <md-list-item v-if="!domain" @click="replaceSearchText(/type:\s*/i, 'type:TOTALRP3')">Total RP</md-list-item>
        <md-list-item v-if="domain === 1" @click="replaceSearchText(/type:\s*/i, 'type:DELVUI')">DelvUI</md-list-item>
      </md-list>
      <md-list v-else-if="dropdownMenu === 'date'">
        <md-list-item><h4>{{ $t('Select Date') }} <span class="syntax">{{ $t('YYYY-MM-DD') }}</span><span class="close-search" @click="clearSearch(/date:\s*/i)">✖</span></h4></md-list-item>
        <md-button-toggle md-single class="md-accent md-warn">
          <md-button :class="{ 'md-toggle': dateType === 'date' }" @click="dateType='date'">{{ $t('During') }}</md-button>
          <md-button :class="{ 'md-toggle': dateType === 'before' }"@click="dateType='before'">{{ $t('Before') }}</md-button>
          <md-button :class="{ 'md-toggle': dateType === 'after' }"@click="dateType='after'">{{ $t('After') }}</md-button>
        </md-button-toggle>
        <datepicker v-model="selectedDate" :inline="true" :disabled-dates="datePickerCfg.disabledDates" @selected="addDateField('date')" />
        <template v-if="!domain">
          <md-list-item><h4>{{ $t('Or filter by recent patch') }}</h4></md-list-item>
          <md-list-item @click="replaceSearchText(/(date|during|before|after):\s*/i, `after:2021-06-29`)"><strong>{{ $t('Shadowlands Patch 9.1.0') }}</strong><span>2021-06-29</span></md-list-item>
          <md-list-item @click="replaceSearchText(/(date|during|before|after):\s*/i, `after:2021-06-01`)"><strong>{{ $t('The Burning Crusade Classic Patch 2.5.1') }}</strong><span>2021-06-01</span></md-list-item>
        </template>
      </md-list>
      <md-list v-else-if="dropdownMenu === 'metric'">
        <md-list-item><h4>{{ $t('Configure a metric filter') }}<span class="close-search" @click="clearSearch(/type:\s*/i)">✖</span></h4></md-list-item>
        <md-list-item class="metric-config">
          <select v-model="metricName">
            <option value="Installs" v-if="!domain">{{ $t('Installs') }}</option>
            <option value="Stars">{{ $t('Stars') }}</option>
            <option value="Views">{{ $t('Views') }}</option>
          </select>
          <select v-model="metricComp">
            <option value=">">&gt;</option>
            <option value="<">&lt;</option>
          </select>
          <input type="text" v-model="metricValue" @focus="$event.target.select()" @keyup="watchMetricValue" @input="watchMetricValue">
          <md-button @click="addMetricField()">{{ $t('Add Filter') }}</md-button>
        </md-list-item>
      </md-list>
    </div>
  </div>
</template>

<script>
import Quill from 'quill'
const Delta = Quill.import('delta')
const Parchment = Quill.import('parchment')
const Clipboard = Quill.import('modules/clipboard')

import Datepicker from 'vuejs-datepicker'

class PlainClipboard extends Clipboard {
  convert(html = null) {
    if (typeof html === 'string') {
      this.container.innerHTML = html
    }
    let text = this.container.innerText
    this.container.innerHTML = ''
    return new Delta().insert(text)
  }
}
Quill.register('modules/clipboard', PlainClipboard, true)

class SearchTag extends Parchment.Embed {
  static create(tag) {
    const node = super.create(tag.text)
    node.innerText = '  '+tag.text
    if (tag.class) {
      tag.class.split(/\s+/g).forEach(c => {
        node.classList.add(c)
      })
    }
    if (tag.search) {
      node.setAttribute('data-search', tag.search)
    }
    node.contentEditable = 'false'
    this._addRemovalButton(node)
    return node
  }

  static value(node) {
    return node.childNodes[0].textContent
  }

  static _addRemovalButton(node) {
    const button = document.createElement('button')
    button.innerHTML = '<span>✖</span>    '
    button.onclick = () => node.remove()
    button.contentEditable = 'false'
    node.appendChild(button)
  }
}
SearchTag.blotName = 'tag'
SearchTag.tagName = 'SPAN'
SearchTag.className = 'search-tag'
Quill.register(SearchTag)

export default {
  components: {
    Datepicker
  },
  data: function () {
    return {
      quill: null,
      searchMode: 'standard',
      showPlaceholder: true,
      inputHasTag: false,
      searchID: 'search' + Math.random().toString(36).substring(7),
      isShowingDropdown: false,
      dropdownMenu: '',
      exampleDate: (new Date(Date.now() - 604800000)).toISOString().split(/T/)[0],
      selectedDate: '',
      dateReplace: null,
      datePickerCfg: {
        disabledDates: {
          from: (new Date(Date.now()))
        }
      },
      dateType: 'date',
      metricName: 'Stars',
      metricComp: '>',
      metricValue: 10
    }
  },
  computed: {
    watchSiteSearch () {
      return this.$store.state.siteSearch
    },
    betaUser () {
      return this.$store.state.user && this.$store.state.user.access && this.$store.state.user.access.beta
    },
    domain () {
      return parseInt(this.$store.state.domain || '0')
    },
    typeFilters () {
      if (this.domain === 1) { // FF14
        return [
          {regex: /(.*)(type:\s?DelvUI)(.*)/i, name: 'DelvUI', class:'imptype type-delvui', search: 'type:delvui'},
        ]
      }
      else { // WoW
        return [
          {regex: /(.*)(type:\s?WeakAura)(.*)/i, name: 'WeakAura', class:'imptype type-wa', search: 'type:weakaura'},
          {regex: /(.*)(type:\s?Plater)(.*)/i, name: 'Plater', class:'imptype type-plater', search: 'type:plater'},
          {regex: /(.*)(type:\s?ElvUI)(.*)/i, name: 'ElvUI', class:'imptype type-elvui', search: 'type:elvui'},
          {regex: /(.*)(type:\s?VuhDo)(.*)/i, name: 'VuhDo', class:'imptype type-vuhdo', search: 'type:vuhdo'},
          {regex: /(.*)(type:\s?OPie)(.*)/i, name: 'OPie', class:'imptype type-opie', search: 'type:opie'},
          {regex: /(.*)(type:\s?TotalRP3?)(.*)/i, name: 'Total RP', class:'imptype type-totalrp', search: 'type:totalrp3'},
          {regex: /(.*)(type:\s?Collection?)(.*)/i, name: 'Collection', class:'imptype type-collection', search: 'type:collection'},
          {regex: /(.*)(type:\s?Snippet?)(.*)/i, name: 'Snippet', class:'imptype type-snippet', search: 'type:snippet'},
        ]
      }
    },
    expansionFilters () {
      if (this.domain === 1) {
        return []
      }
      else {
        return [
          {regex: /(.*)(expansion:\s?(:?sl|shadowlands))(.*)/i, name: 'Shadowlands', class:'expansion exp-sl', search: 'expansion:sl'},
          {regex: /(.*)(expansion:\s?bfa|battle for azeroth)(.*)/i, name: 'Battle for Azeroth', class:'expansion exp-bfa', search: 'expansion:bfa'},
          {regex: /(.*)(expansion:\s?legion)(.*)/i, name: 'Legion', class:'expansion exp-legion', search: 'expansion:legion'},
          {regex: /(.*)(expansion:\s?classic)(.*)/i, name: 'Classic', class:'expansion exp-classic', search: 'expansion:classic'},
          {regex: /(.*)(expansion:\s?(:?t?bcc?|tbc classic))(.*)/i, name: 'TBC Classic', class:'expansion exp-tbc', search: 'expansion:tbc'},
        ]
      }
    }
  },
  watch: {
    watchSiteSearch (value) {
      if (!value) {
        this.searchMode = 'standard'
      }
      this.quill.setContents([{ insert: `${value} \n` }])
      for (let i = 0; i < (value.match(/!|:/g) || []).length; i++) {
        this.parseSearchString()
      }
      this.quill.focus()
      this.dropdownMenu = ''
    },
    domain (v, x) {
      this.quill.setContents([{insert: '\n'}])
      this.quill.focus()
    }
  },
  methods: {
    watchMetricValue (e) {
      if (this.metricValue && this.metricValue.match(/[^\d]/)) {
        this.metricValue = this.metricValue.replace(/[^\d]/g, '')
      }
      if (e.keyCode === 13) {
        this.addMetricField()
      }
    },
    parseSearchString: function (d, o, source) {
      if (source === 'api') {
        return
      }
      const contents = this.quill.getContents()
      if (contents.ops.length === 1 && typeof contents.ops[0].insert === 'string' && contents.ops[0].insert.match(/\s{2,}/) && !contents.ops[0].insert.match(/\w/)) {
        this.quill.setContents([{ insert: `\n` }])
        this.showPlaceholder = true
        this.quill.focus()
        this.resize()
        return
      }
      let hasChanges = false
      const newContentOps = []
      let delta = new Delta()
      let openMenu = false

      this.inputHasTag = false
      this.showPlaceholder = (contents.ops.length === 0 || (contents.ops.length === 1 && contents.ops[0] && contents.ops[0].insert === '\n'))
      for (let i = 0; i < contents.ops.length; i++) {
        if (typeof contents.ops[i].insert === 'string') {
          if (contents.ops[i].insert.match(/!(code|mentions|starred)!/i)) {
            let m = contents.ops[i].insert.match(/(.*)(!(code|mentions|starred)!)(.*)/i)
            if (m) {
              hasChanges = true
              delta = delta
                .retain(m[1].length)
                .delete(m[2].length)
                .retain(m[4].length)
              this.searchMode = m[3]
            }
          }

          else if (contents.ops[i].insert.match(/expansion:/i)) {
            let hasExpansion = false
            for (let exp of this.expansionFilters) {
              let m = contents.ops[i].insert.match(exp.regex)
              if (m) {
                hasChanges = true
                this.inputHasTag = true
                openMenu = true
                hasExpansion = true
                delta = delta
                  .retain(m[1].length)
                  .delete(m[2].length)
                  .insert({tag: {text: exp.name, class: exp.class, search: exp.search}})
                  .retain(m[3].length)
                break
              }
              if (!hasExpansion) {
                this.dropdownMenu = 'expansion'
              }
            }
          }

          else if (contents.ops[i].insert.match(/type:/i)) {
            let hasType = false
            for (let exp of this.typeFilters) {
              let m = contents.ops[i].insert.match(exp.regex)
              if (m) {
                hasChanges = true
                this.inputHasTag = true
                openMenu = true
                hasType = true
                delta = delta
                  .retain(m[1].length)
                  .delete(m[2].length)
                  .insert({tag: {text: exp.name, class: exp.class, search: exp.search}})
                  .retain(m[3].length)
                break
              }
              if (!hasType) {
                this.dropdownMenu = 'type'
              }
            }
          }

          else if (contents.ops[i].insert.match(/collection:/i)) {
            let m = contents.ops[i].insert.match(/(.*)(collection:\s?([\w-]{7,14}))(.*)/i)
            let hasType = false
            if (m) {
              hasChanges = true
              this.inputHasTag = true
              openMenu = true
              hasType = true
              delta = delta
                .retain(m[1].length)
                .delete(m[2].length)
                .insert({tag: {text: `📦 ${m[3]}`, class: 'tag-collection', search: `collection:${m[3]}`}})
                .retain(m[4].length)
              break
            }
          }

          else if (contents.ops[i].insert.match(/user:\s?"/i)) {
            let m = contents.ops[i].insert.match(/(.*)(user:\s?"(.*)")(.*)/i)
            let hasType = false
            if (m) {
              hasChanges = true
              this.inputHasTag = true
              openMenu = true
              hasType = true
              delta = delta
                .retain(m[1].length)
                .delete(m[2].length)
                .insert({tag: {text: `${m[3]}`, class: 'tag-user', search: `user:"${m[3]}"`}})
                .retain(m[4].length)
              break
            }
          }

          else if (this.searchMode === 'standard' && contents.ops[i].insert.match(/(category|tag):/i)) {
            let str = this.getRawSearch()
            let type, game
            let m = str.match(/type:(\w+)/i)
            if (m) {
              type = m[1].toUpperCase()
            }
            m = str.match(/expansion:(\w+)/i)
            if (m) {
              game = m[1]
            }
            let hasCategory = false
            let cats = window.Categories.getCategories(/\w/, false, type, game)
            let bestMatch = ['', '', '']
            let bestMatchCat
            for (let cat of cats) {
              let re = new RegExp(`(.*)((?:category|tag):\\s?(?:${cat.id}|${cat.text}))(.*)`, 'i')
              let m = contents.ops[i].insert.match(re)
              if (m && m[2].length > bestMatch[2].length) {
                bestMatch = m
                bestMatchCat = cat
              }
            }

            if (bestMatch[2].length) {
              hasChanges = true
              this.inputHasTag = true
              hasCategory = true
              delta = delta
                .retain(bestMatch[1].length)
                .delete(bestMatch[2].length)
                .insert({tag: {text: bestMatchCat.text, class: bestMatchCat.id + ' category', search: `category:${bestMatchCat.id}`}})
                .retain(bestMatch[3].length)
            }
          }

          else if (contents.ops[i].insert.match(/(date|during|before|after):/i)) {
            let m = contents.ops[i].insert.match(/(.*)((date|during|before|after):\s?(\d\d\d\d-\d\d-\d\d))(.*)/i)
            if (m) {
              hasChanges = true
              this.inputHasTag = true
              openMenu = true
              let dateType = 'date'
              if (m[3].toLowerCase() === 'before') {
                dateType = 'before'
              }
              else if (m[3].toLowerCase() === 'after') {
                dateType = 'after'
              }
              delta = delta
                .retain(m[1].length)
                .delete(m[2].length)
                .insert({tag: {text: `${dateType[0].toUpperCase() + dateType.substring(1)}: ${m[4]}`, class: 'filter-date', search: `${dateType}:${m[4]}`}}).insert(' ')
              break
            }
            else {
              if (contents.ops[i].insert.match(/(date|during):/i)) {
                this.dateType = 'date'
              }
              else if (contents.ops[i].insert.match(/before:/i)) {
                this.dateType = 'before'
              }
              else if (contents.ops[i].insert.match(/after:/i)) {
                this.dateType = 'after'
              }
              this.dropdownMenu = 'date'
            }
          }

          else if (contents.ops[i].insert.match(/metric:/i)) {
            let m = contents.ops[i].insert.match(/(.*)(metric:\s?(installs|stars|views)(<|>)(\d+))(.*)/i)
            if (m) {
              hasChanges = true
              this.inputHasTag = true
              openMenu = true
              let metric = m[3].charAt(0).toUpperCase() + m[3].slice(1).toLowerCase()
              delta = delta
                .retain(m[1].length)
                .delete(m[2].length)
                .insert({tag: {text: `${metric} ${m[4]} ${m[5]}`, class: 'filter-metric', search: `metric:${m[3].toLowerCase()}${m[4]}${m[5]}`}})
                .retain(m[6].length)
              break
            }
            else {
              this.dropdownMenu = 'metric'
            }
          }

          else {
            delta = delta.retain(contents.ops[i].insert.length)
          }
        }

        else if (typeof contents.ops[i].insert !== 'string') {
          this.inputHasTag = true
          delta = delta.retain(1)
        }
      }

      if (delta.ops.length) {
        this.quill.updateContents(delta)
      }
      if (openMenu && source === 'user') {
        this.openDropdownMenu(true)
      }
      if (hasChanges) {
        this.parseSearchString()
      }
      this.resize()
    },
    resize: function () {
      const searchbox = document.querySelector(`#${this.searchID} .ql-editor p`).getBoundingClientRect()
      const topbar = document.getElementById('topbar').getBoundingClientRect()

      this.$refs.container.style.width = Math.min(searchbox.width + 160, topbar.width - 590) + 'px'
    },
    addDateField: function () {
      this.$nextTick(() => {
        if (this.selectedDate) {
          this.replaceSearchText(new RegExp(`(date|during|before|after):\\s*`, 'i'), `${this.dateType}:${this.selectedDate.toISOString().split(/T/)[0]}`)
          this.selectedDate = null
          this.dropdownMenu = 'main'
        }
      })
    },
    addMetricField: function () {
      const num = parseInt(this.metricValue)
      const str = `metric:${this.metricName}${this.metricComp}${num}`
      if (num > 0) {
        this.$nextTick(() => {
          this.replaceSearchText(new RegExp(`metric:\\s*`, 'i'), str)
          this.dropdownMenu = 'main'
        })
      }
    },
    setSearchMode: function (mode) {
      this.searchMode = mode
      this.quill.setContents([{ insert: '\n' }])
      this.quill.focus()
    },
    addSearchText: function (text) {
      let selection = this.quill.getSelection(true)
      if (selection) {
        this.quill.insertText(selection.index || 0, text)
      }
      else {
        let delta = new Delta().insert(text)
        this.quill.setContents(delta)
      }
      this.parseSearchString()
    },
    replaceSearchText: function (find, replace) {
      const contents = this.quill.getContents()
      let delta = new Delta()
      for (let i = 0; i < contents.ops.length; i++) {
        if (typeof contents.ops[i].insert === 'string') {
          let m = contents.ops[i].insert.match(new RegExp(`(.*)(${find.source})(.*)`, 'i'))
          if (m) {
            delta = delta.retain(m[1].length).delete(m[2].length).insert(replace).retain(m[3].length)
          }
          else {
            delta = delta.retain(contents.ops[i].insert.length)
          }
        }
        else {
          delta = delta.retain(1)
        }
      }
      this.quill.updateContents(delta)
      this.parseSearchString()
      this.dropdownMenu = 'main'
    },
    checkSearchSubmit: function (e) {
      if (e.keyCode === 13) {
        e.preventDefault()
        this.submitSearch()
      }
    },
    getRawSearch: function () {
      const html = this.$refs.searchInput.innerHTML
      const el = document.createElement('div')
      el.innerHTML = this.$refs.searchInput.innerHTML
      const search = el.querySelector('.ql-editor p')

      let str = ''
      let betaUser = false
      if (this.searchMode !== 'standard') {
        str = `!${this.searchMode}!`
      }
      search.childNodes.forEach(node => {
        if (node.nodeName === '#text') {
          str = `${str} ${node.nodeValue}`
        }
        else if (node.nodeName === 'SPAN' && node.dataset && node.dataset.search) {
          str = `${str} ${node.dataset.search}`
        }
      })
      return str
    },
    submitSearch: function () {
      const str = this.getRawSearch().trim()
      if (str) {
        this.$store.commit('setSearchText', str, true)
        this.$router.push(`/search/${str.replace(/\s+/g, '%20')}`)
      }
      this.quill.focus()
    },
    clearSearch: function (clear) {
      this.replaceSearchText(clear, '')
      this.dropdownMenu = 'main'
    },
    openDropdownMenu: function (reset) {
      if (!this.dropdownMenu || reset) {
        this.dropdownMenu = 'main'
      }
      this.quill.focus()
    },
    update: function (e) {
      this.parseSearchString()
    },
    watchScroll: function () {
      const el = this.$refs.searchDropdown
      if (el) {
        const rect = el.getBoundingClientRect()
        if (rect.height + rect.top < 0) {
          this.dropdownMenu = ''
        }
      }
    }
  },
  mounted: function () {
    this.quill = new Quill(`#${this.searchID}`)
    const kb = this.quill.getModule('keyboard')
    delete kb.bindings[13] // disable line breaks
    this.quill.on('text-change', this.parseSearchString)
    this.parseSearchString()
  },
  created: function () {
    window.addEventListener('scroll', this.watchScroll)
    window.addEventListener('resize', this.resize)
  },
  destroyed: function () {
    window.removeEventListener('scroll', this.watchScroll)
    window.removeEventListener('resize', this.resize)
  },
}
</script>



<style lang="scss">
.search-container {
  position: relative;
  max-width: 100%;
  min-width: 430px;
  width: 50%;
  height:40px;
  background:#404040;
  padding: 10px;
  margin-left: 16px;
  border-radius: 4px;

  .search-icon {
    position: absolute;
    top:8px;
    right:10px;
    opacity: .5;
    border-radius: 8px;
    &:hover {
      cursor: pointer;
      background: #555;
    }
  }
  .search-input {
    position: absolute;
    top:0;
    left:0;
    width: 100%;
    height: 100%;
    line-height: 40px;
    margin-right: 52px;
    outline: 0;
    color: #EEE;
    white-space: nowrap;
    overflow: hidden;

    &:before {
      color: #CCC;
      background: #5A3A3C;
      padding: 0 8px;
      border-radius: 4px;
      border-right: 1px solid black;
      line-height: 40px;
      position: absolute;
    }

    &.standard-search {
      width: calc(100% - 40px);
      &:before {
        content: 'Search';
      }
      .ql-editor {
        padding-left: 66px;
      }
    }
    &.code-search {
      width: calc(100% - 30px - 40px);
      &:before {
        content: '[Beta] Code';
      }
      .ql-editor {
        padding-left: 96px; /*56px*/
      }
    }

    &.mentions-search {
      width: calc(100% - 58px);
      &:before {
        content: 'Mentions';
      }
      .ql-editor {
        padding-left: 84px;
      }
    }

    &.starred-search {
      width: calc(100% - 44px);
      &:before {
        content: 'Starred';
      }
      .ql-editor {
        padding-left: 70px;
      }
    }

    .ql-editor {
      margin: 0;
      padding: 0;
      white-space: nowrap;
      overflow: hidden;
      outline: 0;
      width: 100%;
      height: 100%;
      line-height: inherit;
      p {
        margin: 0;
        display: inline;
        white-space: pre;
      }
    }

    .search-tag {
      display: inline-block;
      margin-top: -4px;
      background: #212121;
      border: 1px solid;
      color: #C79C6E;
      border-radius: 8px;
      padding: 2px 6px 2px 0;
      white-space: pre-wrap;
      margin: 0 4px;
      line-height: 14px;
      cursor: default;
      &.imptype {
        border-color: #fff0fd;
        color: #fff0fd;
      }
      &.expansion {
        border-color: #d8652e;
        color: #d8652e;
      }
      &.tag-collection {
        color: #CAA27E;
        border-color: #CAA27E;
      }
      &.tag-user {
        color: #FFC83D;
        border-color: #FFC83D;
      }
      &.exp-sl {
        border-color: #eaae27;
        color: #eaae27;
      }
      &.exp-tbc {
        border-color: #BED82E;
        color: #BED82E;
      }
      &.filter-date {
        border-color: #e2fffa;
        color: #e2fffa;
      }
      &.filter-metric {
        border-color: #9fecd0;
        color: #9fecd0;
      }
      button {
        padding: 0;
        font-size: 12px;
        margin-left: 4px;
        width: 14px;
        color: inherit;
        border: 0;
        background: transparent;
        cursor: pointer;
        white-space: pre;
        span {
          padding: 2px;
          width: 14px;
          border-radius: 6px;
        }
        span:hover {
          background: #000000;
        }
      }
      span {
        display: inline-block;
      }
    }
  }
}

.search-dropdown-block {
  position: absolute;
  overflow: hidden;
  top: 0; left: 0; right: 0; bottom: 0;
  min-height: 100vh; height: 100%;
}

#topbar .search-dropdown {
  left: 128px;
}

#mobile-sidebar .search-dropdown {
  left: 16px;
}

.search-dropdown {
  position: absolute;
  top: 63px;
  width: 90%; max-width: 400px;
  z-index: 9999999;
  border: 1px solid #040404;
  &.hidden {
    display: none;
  }
  .md-button-toggle {
    justify-content: space-around;
  }
  .md-list {
    margin: 0!important;
    .md-list-item {
      h4 {
        margin: 0;
        font-size: 14px;
        text-transform: uppercase;
        border-bottom: 1px solid #777;
        margin: 8px 0;
        width: 100%;
        color: #c0272d;
        .close-search {
          float: right;
          cursor: pointer;
        }
      }
      .md-list-item-container {
        justify-content: normal;
        min-height: auto;
        font-size: 14px;
        strong {
          &:after {
            content: ':';
          }
        }
        span {
          margin-left: 16px; color: #888;
        }
        em {
          font-size: 12px;
          &:before {content: '* '}
        }
      }
    }
    &:after {
      content: none;
    }
    .metric-config {
      select, input {
        background: inherit;
        color: inherit;
        padding: 4px;
        line-height: 16px;
        margin-right: 8px;
        border: 1px solid #767676;

      }
      select option {
        background: #333333;
      }
      input {
        width: 50px;
        padding-top: 5px;
        outline: none;
        text-align: right;
      }
    }
  }

  .syntax {
    text-transform: none;
    font-size: 90%;
  }

  .vdp-datepicker .vdp-datepicker__calendar {
    background: inherit;
    border: none;
    padding: 0 16px;
    width: auto;

    .prev {
      &:after {
        border-right: 10px solid #c0272d;
      }
      &.disabled:after {
        border-left: 10px solid #666;
      }
      &:not(.disabled):hover {
        background: #77777733;
      }
    }
    .next {
      &:after {
        border-left: 10px solid #c0272d;
      }
      &.disabled:after {
        border-left: 10px solid #666;
      }
      &:not(.disabled):hover {
        background: #77777733;
      }
    }
    .up {
      &:not(.disabled):hover {
        background: #77777733;
      }
    }
    .cell {
      &.today {
        background: #77777711;
      }
      &.selected {
        border: 1px solid #c0272d!important;
        background: inherit;
      }
      &:not(.blank):not(.disabled).day:hover, &:not(.blank):not(.disabled).month:hover, &:not(.blank):not(.disabled).year:hover {
        border: 1px solid #212121;
        background: #77777733;
      }
    }
    .cell.day.disabled {
      color: #666;
    }

  }
}



</style>