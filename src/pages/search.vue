<template lang="">
  
<main>
  <section class="section" v-infinite-scroll="loadMore" infinite-scroll-disabled="loading">
    <div class="container">
      <div class="level">
        <div class="level-left">
          <h2 class="title is-4">Search for <strong>{{ query }}</strong></h2>
        </div>
        <div class="level-right">
          <div class="control is-grouped">
            <v-select @option-selected="changeOrder" :options="orderOptions" :default="order" aria-label="order"></v-select>
            <v-select @option-selected="changeTimeframe" :options="timeframeOptions" :default="timeframe" aria-label="timeframe"></v-select>
            <div class="tooltip">
              <v-select @option-selected="changeLanguage" :options="languageOptions" disabled placeholder="Language" aria-label="language"></v-select><span class="tooltip-text">Not work only because YouTube API <a href="https://code.google.com/p/gdata-issues/issues/detail?id=4110&amp;q=relevanceLanguage&amp;colspec=API%20ID%20Type%20Status%20Priority%20Stars%20Summary" target="_blank" rel="noopener">issue</a></span>
            </div>
          </div>
        </div>
      </div>
      <video-list :videos="results"></video-list>
      <spinner :show="loading"></spinner>
      <div class="no-results" v-show="noResults"> 
        <router-link to="/"> <img :src="logo" :alt="appName + ' logo'"></router-link><span>No results.</span>
      </div>
    </div>
  </section>
</main>
</template>

<script>
import filters from '~api/filters'
import store from '~store'
import config from '~config'
import vSelect from '~components/Select.vue'
import Spinner from '~components/Spinner.vue'
import VideoList from '~components/VideoList.vue'

export default {
  components: { vSelect, VideoList, Spinner },
  data() {
    return {
      orderOptions: filters.order,
      languageOptions: filters.language,
      timeframeOptions: filters.timeframe,
      results: [],
      language: '',
      timeframe: filters.timeframe[0].value,
      order: filters.order[0].value,
      limit: 28,
      loading: false,
      nextPage: null,
      noResults: false,
      params: {
        title: ''
      }
    }
  },
  computed: {
    query() {
      return this.$route.query.q
    },
    logo() {
      return config.app.brand.logoMini
    },
    appName() {
      return config.app.name
    }
  },
  head: {
    title() {
      return {
        inner: this.params.title,
        separator: '-',
        complement: config.app.name
      }
    },
    meta() {
      return [
        { id: 'description', name: 'description', content: 'Site search' }
      ]
    }
  },
  created() {
    this.$Progress.start()
    this.nextPage = null
    this.loadResults()
  },
  watch: {
    order() {
      this.results = []
      this.nextPage = null
      this.loadResults()
    },
    language() {
      this.results = []
      this.nextPage = null
      this.loadResults()
    },
    timeframe() {
      this.results = []
      this.nextPage = null
      this.loadResults()
    },
    $route() {
      this.results = []
      this.nextPage = null
      this.loadResults()
    }
  },
  methods: {
    changeOrder(order) {
      this.order = order
    },
    changeLanguage(language) {
      this.language = language
    },
    changeTimeframe(timeframe) {
      this.timeframe = timeframe
    },
    loadResults() {
      this.noResults = false
      this.loading = true
      this.params.title = this.query
      this.$emit('updateHead')

      const params = {
        q: '($q)'.replace(/\$q/g, this.query),
        order: this.order,
        maxResults: this.limit,
        videoCategoryId: 27,
        pageToken: this.nextPage,
        publishedAfter: this.timeframe || null,
        relevanceLanguage: this.language || null
      }

      store.fetchVideos(params).then((result) => {
        this.loading = false
        this.$Progress.finish()
        if(!result.videos.length) {
          this.noResults = (!this.results.length)
          this.nextPage = null
          return
        }
        this.nextPage = result.nextPageToken
        result.videos.map(video => {
          this.results.push(video)
        })
      }).catch(() => {
        this.loading = false
        this.$Progress.fail()
      })
    },
    loadMore() {
      if(!this.nextPage) { return }
      this.loadResults()
    }
  }
}
</script>

<style lang="sass" scoped>
.no-results
  width: 100px
  margin: 0 auto
  text-align: center
  img
    float: left
    display: inline-block
    width: 100%
    height: 30px
    margin-bottom: 10px
  span
    display: inline-block

.tooltip
  position: relative
  display: inline-block
  &:hover
    .tooltip-text
      opacity: 1

  .tooltip-text
    transition: opacity .2s ease-in-out
    opacity: 0
    background-color: rgba(0,0,0,.85)
    color: #fff
    text-align: center
    padding: 5px
    border-radius: 4px
    position: absolute
    z-index: 1

</style>