<template>
    <div class="city-list" ref="wrapper">
        <div>
            <div class="area">
                <div class="city-title">当前城市</div>
                <div class="city-btn-list">
                    <div class="city-btn-item">
                        <div class="button">{{this.currentCity}}</div>
                    </div>
                </div>
            </div>
            <div class="area">
                <div class="city-title">热门城市</div>
                <div class="city-btn-list">
                    <div class="city-btn-item"
                        v-for="item in hotCities"
                        :key="item.id"
                        @click="handleCityClick(item.name)"
                    >
                        <div class="button">{{item.name}}</div>
                    </div>
                </div>
            </div>
            <div class="area"
                v-for="(item, key) in cities"
                :key="key"
                :ref="key"
            >
                <div class="city-title">{{key}}</div>
                <div class="city-alphbet-list">
                    <div class="city-alphbet-item border-topbottom"
                        v-for="innerItem in item"
                        :key="innerItem.id"
                        @click="handleCityClick(innerItem.name)"
                    >
                        {{innerItem.name}}
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import BScroll from 'better-scroll'
import { mapState, mapMutations } from 'vuex'
export default {
  name: 'CityList',
  props: {
    'hotCities': Array,
    'cities': Object,
    'letter': String
  },
  mounted () {
    this.$nextTick(() => {
      this.scroll = new BScroll(this.$refs.wrapper, {})
    })
  },
  methods: {
    handleCityClick (city) {
      this.cityChange(city)
      this.$router.push('/')
    },
    ...mapMutations(['cityChange'])
  },
  computed: {
    ...mapState({
      currentCity: 'city'
    })
  },
  watch: {
    letter () {
      if (this.letter) {
        const ele = this.$refs[this.letter][0]
        this.scroll.scrollToElement(ele)
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
  .city-list
    overflow hidden
    position absolute
    top 1.58rem
    left 0
    right 0
    bottom 0
    .city-title
      line-height .54rem
      background #eee
      padding-left .2rem
      color #666666
      font-size .26rem
    .city-btn-list
      overflow hidden
      padding .1rem .6rem .1rem .1rem
      .city-btn-item
        width 33.3%
        float left
        .button
          margin .1rem
          padding .1rem 0
          text-align center
          border .02rem solid #cccccc
          border-radius .06rem
    .city-alphbet-list
      .city-alphbet-item
        line-height .76rem
        padding-left .2rem
</style>
