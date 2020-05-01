<template>
    <div>
        <ul class="alphabet-list">
            <li class="alphabet-item"
              v-for="(item, key) in cities"
              :key="key"
              :ref="key"
              @click="handleLetterClick"
              @touchstart.prevent="handleTouchStart"
              @touchmove="handleTouchMove"
              @touchend="handleTouchend"
            >
              {{key}}
            </li>
        </ul>
    </div>
</template>

<script>
export default {
  name: 'CityAlphabet',
  props: {
    'cities': Object
  },
  data () {
    return {
      touchStatus: false,
      startY: 0,
      timer: null
    }
  },
  methods: {
    handleLetterClick (e) {
      this.$emit('letterChange', e.target.innerText)
    },
    handleTouchStart () {
      this.touchStatus = true
    },
    handleTouchMove (e) {
      if (this.touchStatus) {
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timer = setTimeout(() => {
          const touchY = e.touches[0].clientY - 79
          console.log(touchY)
          const index = Math.floor((touchY - this.startY) / 20)
          if (index >= 0 && index < this.letters.length) {
            this.$emit('letterChange', this.letters[index])
          }
        }, 0)
      }
    },
    handleTouchend () {
      this.touchStatus = false
    }
  },
  computed: {
    letters () {
      const letters = []
      for (let i in this.cities) {
        letters.push(i)
      }
      return letters
    }
  },
  updated () {
    this.startY = this.$refs['A'][0].offsetTop
  }
}
</script>

<style lang="stylus" scoped>
  @import '~@/assets/styles/variables.styl'
  .alphabet-list
    position absolute
    top 1.58rem
    right 0
    bottom 0
    display flex
    flex-direction column
    justify-content center
    width .4rem
    .alphabet-item
      line-height .4rem
      text-align center
      color $bgColor
</style>
