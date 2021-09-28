<template>
  <div class="index-bar-content">
    <el-scrollbar style="height: 100%" ref="scrollbar">
      <div :id="key" class="main-list" v-for="(value, key) in indexData" :key="key" ref="listGroup">
        <div class="title-key">{{ key }}</div>
        <div class="content-container">
          <div class="content-item" v-for="(val, index) in value" :key="index">
            {{ val[name] }}
          </div>
        </div>
      </div>
    </el-scrollbar>
    <!-- 右侧字母列表 -->
    <ul class="char-list">
      <li v-if="totalPage > 1" @click="handlePreviousPage">
        <i class="iconfont iconshang"></i>
      </li>
      <li
        v-for="item in indexList"
        :key="item.id"
        @click="scrollToLetter(item)"
        :class="{ active: currentIndex === item.id }"
      >
        {{ item.key }}
      </li>
      <li v-if="totalPage > 1" @click="handleNextPage">
        <i class="iconfont iconxia"></i>
      </li>
    </ul>
  </div>
</template>

<script>
import pinyin from "js-pinyin"
export default {
  name: "List",
  props: {
    sourceData: {
      type: Array,
      default: () => []
    },
    name: {
      type: String,
      default: "shopName"
    }
  },
  data() {
    return {
      charList: [],
      listHeight: [],
      currentIndex: 0,
      indexPage: 1,
      indexLimit: 14,
      indexData: {}
    }
  },
  computed: {
    totalPage() {
      return Math.ceil(this.charList.length / this.indexLimit)
    },
    indexList() {
      return this.charList.slice(
        (this.indexPage - 1) * this.indexLimit,
        this.indexPage * this.indexLimit
      )
    }
  },
  created() {
    this.getData()
  },
  mounted() {
    this._calculateHeight()
    this.handleScroll()
  },
  methods: {
    getData() {
      pinyin.setOptions({ checkPolyphone: false, charCase: 0 })
      let alphabet = []
      let _charList = []
      for (let i = 0; i < this.sourceData.length; i++) {
        // 获取原数组每一项的 name 值
        let name = this.sourceData[i][this.name]
        // 获取每一个name值第一个字的大写首字母（传入的 name 是中文时默认得到大写字母，name 是英文时按照原字符串输出，可能是小写）
        let initial = pinyin.getCamelChars(name).substring(0, 1).toUpperCase()
        // 给数组每一项增加名为 initial 的 key，值就是第一个字的大写首字母
        this.sourceData[i].initial = initial
        // 获取用于索引的字母
        if (alphabet.indexOf(initial) === -1) {
          alphabet.push(initial)
        }
      }
      // 按字母表顺序排序
      alphabet.sort()

      // 给每个字母增加唯一标识，后面定位时会用到
      for (var i = 0; i < alphabet.length; i++) {
        _charList.push({
          id: i,
          key: alphabet[i]
        })
      }
      this.charList = _charList
      let resultData = {}
      // 将源数据按照首字母分类
      for (let i = 0; i < alphabet.length; i++) {
        resultData[alphabet[i]] = this.sourceData.filter((item) => {
          return item.initial === alphabet[i]
        })
      }
      // 得到最终结果
      this.indexData = resultData
    },
    _calculateHeight() {
      this.listHeight = []
      this.$nextTick(() => {
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        if (list) {
          for (let i = 0; i < list.length; i++) {
            let item = list[i]
            height += item.clientHeight
            this.listHeight.push(height)
          }
        }
      })
    },
    scrollToLetter(item) {
      let scrollEle = this.$refs.scrollbar.wrap
      scrollEle.scrollTop = this.listHeight[item.id]

      // let element = document.querySelector(`#${item.key}`)
      // console.log(element, "element")
      // element.scrollIntoView({
      //   behavior: "smooth"
      // })
    },
    handleScroll() {
      let scrollEle = this.$refs.scrollbar.wrap
      scrollEle.onscroll = () => {
        let newY = scrollEle.scrollTop
        const listHeight = this.listHeight
        // 在中间部分滚动
        for (let i = 0; i < listHeight.length - 1; i++) {
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          if (height1 <= newY && newY < height2) {
            this.currentIndex = i
            // console.log(this.currentIndex, "currentIndex")
            let currentPage = Math.floor(i / this.indexLimit) + 1
            this.indexPage = currentPage
            return
          }
        }
      }
    },
    handleNextPage() {
      if (this.indexPage < this.totalPage) {
        this.indexPage++
      }
    },
    handlePreviousPage() {
      if (this.indexPage > 1) {
        this.indexPage--
      }
    }
  }
}
</script>

<style lang="scss">
.index-bar-content {
  position: relative;
  width: 400px;
  height: 305px;
  .el-scrollbar__wrap {
    overflow-x: hidden;
    .el-scrollbar__view {
      padding: 0 20px;
    }
  }
  .main-list {
    padding-top: 10px;
    .title-key {
      padding-bottom: 12px;
      font-size: 14px;
      font-weight: bold;
    }
    .content-container {
      display: flex;
      flex-wrap: wrap;
      .content-item {
        margin-right: 16px;
        margin-bottom: 12px;
        font-size: 12px;
      }
    }
  }
}
.el-popover {
  padding: 0;
}
.char-list {
  z-index: 99;
  width: 24px;
  height: 100%;
  background: #fafbfe;
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  list-style: none;
  display: flex;
  flex-direction: column;
  text-align: center;
  li {
    height: 19px;
    display: inline;
    cursor: pointer;
    font-size: 14px;
  }
  .active {
    color: #fd4378;
  }
}
</style>