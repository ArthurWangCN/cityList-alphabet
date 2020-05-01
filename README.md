

> 移动端SPA如旅游类或团购类经常用到城市页，用 vue 封装了一个城市页，包括城市搜索、城市列表选择、右侧按字母滚动到相应城市等功能



代码文件结构：

```
City.vue
	|--- CityHeader.vue
	|--- CitySearch.vue
	|--- CityList.vue
	|--- CityAlphabet.vue
```



**City.vue —— 主文件：**

```javascript
<template>
  <div>
    <city-header></city-header>
    <city-search :cities="cities"></city-search>
    <city-list
      :hotCities="hotCities"
      :cities="cities"
      :letter="letter"
    ></city-list>
    <city-alphabet
      :cities="cities"
      @letterChange="handleLetterChange"
    ></city-alphabet>
  </div>
</template>

<script>
import axios from 'axios'
import CityHeader from './components/CityHeader'
import CitySearch from './components/CitySearch'
import CityList from './components/CityList'
import CityAlphabet from './components/CityAlphabet'

export default {
  name: 'City',
  data () {
    return {
      hotCities: [],	// 热门城市
      cities: {},	// 城市列表
      letter: ''	// 选择的字母
    }
  },
  components: {
    CityHeader,
    CitySearch,
    CityList,
    CityAlphabet
  },
  mounted () {
    this.getCityInfo()
  },
  methods: {
    // 获取城市列表
    getCityInfo () {
      // 这里是mock数据，项目中更换为真实城市列表接口
      axios.get('/mock/city.json').then(this.getCityInfoSucc)
    },
    getCityInfoSucc (res) {
      res = res.data
      if (res.ret && res.data) {
        const data = res.data
        this.hotCities = data.hotCities
        this.cities = data.cities
      }
    },
    handleLetterChange (letter) {
      this.letter = letter
    }
  }
}
</script>
```



**CityHeader.vue —— 简单的头部：**

```html
<template>
    <div>
        <div class="city-header">
            城市列表
            <router-link to="/">	// 返回按钮，点击返回首页
                <div class="iconfont city-icon-back">&#xe624;</div>
            </router-link>
        </div>
    </div>
</template>
```



**CitySearch.vue —— 城市搜索框：**

```javascript
<template>
  <div>
    // 搜索输入框
    <div class="city-search">
      <input type="text" v-model="keyword" class="city-search-input" placeholder="输入城市名或拼音">
    </div>
    // 搜索内容展示
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
      keyword: '',  // 搜索关键字
      list: [],     // 搜索出来匹配城市的列表
      timer: null
    }
  },
  mounted () {
    // 初始化一个better-scroll
    this.scroll = new Bscroll(this.$refs.search)
  },
  methods: {
    // 点击城市进行选择，具体在vuex中实现
    handleCityClick (city) {
      this.cityChange(city)
      this.$router.push('/')
    },
    ...mapMutations(['cityChange'])
  },
  computed: {
    // 搜索无匹配城市
    hasNoData () {
      return !this.list.length
    }
  },
  watch: {
    // 监听搜索keyword，并且通过事件节流来提高性能
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
        // 遍历城市列表，城市名name或城市拼写spell符合的话就保存到数组list中
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
  }
}
</script>
```



vuex-state.js

```javascript
// 默认城市，业务中可以拿定位数据
let defaultCity = '成都'
try {
  // 如果本地存储有城市数据，替换
  if (localStorage.city) {
    defaultCity = localStorage.city
  }
} catch (error) {
    console.log(error)
}
export default {
  city: defaultCity
}
```

vuex-mutations.js

```javascript
export default {
  // 城市切换事件
  cityChange (state, city) {
    state.city = city
    try {
      localStorage.city = city
    } catch (error) {
    }
  }
}
```



**CityList.vue —— 城市列表**

```javascript
<template>
  <div class="city-list" ref="wrapper">
    <div>
      <!-- 当前城市 -->
      <div class="area">
        <div class="city-title">当前城市</div>
        <div class="city-btn-list">
          <div class="city-btn-item">
            <div class="button">{{this.currentCity}}</div>
          </div>
        </div>
      </div>
      <!-- 热门城市 遍历hotCities -->
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
      <!-- 城市列表 遍历cities -->
      <div class="area"
        v-for="(item, key) in cities"
        :key="key"
        :ref="key"
      >
        <!-- 字母类目 如A、B、C -->
        <div class="city-title">{{key}}</div>
        <!-- 字母类目下的城市名 第二层遍历 -->
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
      // 初始化 better-scroll
      this.scroll = new BScroll(this.$refs.wrapper, {})
    })
  },
  methods: {
    // 点击城市切换
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
    // 监听letter，如果改变滚动到相应的位置
    letter () {
      if (this.letter) {
        const ele = this.$refs[this.letter][0]
        this.scroll.scrollToElement(ele)
      }
    }
  }
}
</script>
```



**CityAlphabet.vue**

```javascript
<template>
  <div>
    <ul class="alphabet-list">
      <!-- 遍历cities的key 即A、B、C... -->
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
      touchStatus: false,   // 触摸状态
      startY: 0,    // 默认y方向的值0
      timer: null
    }
  },
  methods: {
    // 点击字母派发letterchange事件
    handleLetterClick (e) {
      this.$emit('letterChange', e.target.innerText)
    },
    // 触摸开始 状态设置为true
    handleTouchStart () {
      this.touchStatus = true
    },
    // 触摸过程 使用事件节流提高效率
    handleTouchMove (e) {
      if (this.touchStatus) {
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timer = setTimeout(() => {
          // touchY 相对于字母表ul y 方向的位置
          // 79为city-header和city-search的高度之和
          const touchY = e.touches[0].clientY - 79
          // index 当前索引 20为每个字母的高度
          const index = Math.floor((touchY - this.startY) / 20)
          // 边界值 在边界内才派发字母改变事件
          if (index >= 0 && index < this.letters.length) {
            this.$emit('letterChange', this.letters[index])
          }
        }, 100)
      }
    },
    // 触摸结束 状态设置为false
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
    // 更新 startY 的值
    this.startY = this.$refs['A'][0].offsetTop
  }
}
</script>
```

