<template>
    <div>
        <div class="city-header">
            城市列表
            <router-link to="/">
                <div class="iconfont city-icon-back">&#xe624;</div>
            </router-link>
        </div>
    </div>
</template>

<script>
export default {
  name: 'CityHeader'
}
</script>

<style lang="stylus" scoped>
    @import '~@/assets/styles/variables.styl'
    .city-header
        position relative
        overflow hidden
        height $headerHeight
        line-height $headerHeight
        text-align center
        color #fff
        font-size .32rem
        background-color $bgColor
        .city-icon-back
            position absolute
            top 0
            left 0
            width .64rem
            text-align center
            font-size .4rem
            color #ffffff
</style>
