<template>
    <div>
        <div class="city-search">
            <input type="text" v-model="keyword" class="city-search-input" placeholder="输入城市名或拼音">
        </div>
        <div class="city-search-content" ref="search" v-show="keyword">
            <ul>
                <li class="city-search-item border-bottom"
                  v-for="item in list"
                  :key="item.id"
                  @click="handleCityClick(item.name)"
                >
                  {{item.name}}
                </li>
                <li class="city-search-item border-bottom" v-show="hasNoData">没有找到匹配数据</li>
            </ul>
        </div>
    </div>
</template>

<script>
import { mapMutations } from 'vuex'
import Bscroll from 'better-scroll'
export default {
  name: 'CitySearch',
  props: {
    'cities': Object
  },
  data () {
    return {
      keyword: '',
      list: [],
      timer: null
    }
  },
  methods: {
    handleCityClick (city) {
      this.cityChange(city)
      this.$router.push('/')
    },
    ...mapMutations(['cityChange'])
  },
  computed: {
    hasNoData () {
      return !this.list.length
    }
  },
  watch: {
    keyword () {
      if (this.timer) {
        clearTimeout(this.timer)
      }
      if (!this.keyword) {
        this.list = []
        return
      }
      this.timer = setTimeout(() => {
        const result = []
        for (let i in this.cities) {
          this.cities[i].forEach((value) => {
            if (value.spell.indexOf(this.keyword) > -1 || value.name.indexOf(this.keyword) > -1) {
              result.push(value)
            }
          })
        }
        this.list = result
      }, 100)
    }
  },
  mounted () {
    this.scroll = new Bscroll(this.$refs.search)
  }
}
</script>

<style lang="stylus" scoped>
  @import '~@/assets/styles/variables.styl'
  .city-search
    height .72rem
    padding 0 .1rem
    background $bgColor
    .city-search-input
        box-sizing border-box
        width 100%
        height .62rem
        line-height .62rem
        padding 0 .1rem
        text-align center
        border-radius .06rem
        color #666
  .city-search-content
    overflow hidden
    position absolute
    top 1.58rem
    left 0
    right 0
    bottom 0
    z-index 9
    background-color #eee
    .city-search-item
      line-height .62rem
      padding-left .2rem
      background #fff
      color #666
</style>
