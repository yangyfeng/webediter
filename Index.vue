<template>
  <div class="yyf-webediter" :style="{height:myConfig.height, width:myConfig.width}" :class="{'ace-full-screen': myConfig.fullScreen}"></div>
</template>
<script>
import brace from 'brace'
import 'brace/ext/language_tools'
// 默认配置
const defaultConfig = {
  width: '400px',
  height: '500px',
  lang: 'text',
  theme: 'idle_fingers',
  readOnly: false,
  fullScreen: false,
  autoCompletion: true,
  tabSize: 2
}
export default {
  props: {
    config: {
      type: Object,
      default () {
        return {}
      }
    },
    value: {
      type: [String, Array, Object],
      default: ''
    }
  },
  data() {
    return {
      myConfig: Object.assign({}, defaultConfig, this.config),
      $ace: null
    }
  },
  mounted() {
    let conf = this.myConfig || defaultConfig
    this.initAce(conf)
  },
  methods: {
    // 判断字符串是否为json数据
    isJsonString(str) {
      try {
        if (typeof JSON.parse(str) === 'object') {
          return true
        }
      } catch (e) {
        console.error('不是一个有效的JSON String')
      }
      return false
    },
    // 设置全屏幕编辑
    setfullScreen() {
      this.$el.classList.toggle('ace-full-screen')
      this.$ace.resize()
    },
    // 初始化Ace
    initAce(conf) {
      this.$ace = brace.edit(this.$el)
      let session = this.$ace.getSession()
      // 加载相应模块
      require(`brace/mode/${conf.lang}`)
      require(`brace/theme/${conf.theme}`)
      // 禁用警告
      this.$ace.$blockScrolling = Infinity
      // 代码提示与自动补全
      this.$ace.setOptions({
        enableBasicAutocompletion: conf.autoCompletion,
        enableLiveAutocompletion: conf.autoCompletion
      })
      // 配置语言
      session.setMode(`ace/mode/${conf.lang}`)
      // 配置主题
      this.$ace.setTheme(`ace/theme/${conf.theme}`)
      // 设置默认内容
      // 是不是json语言
      if (conf.lang === 'json' && typeof this.value === 'object') {
        let newval = JSON.stringify(this.value, null, conf.tabSize)
        this.$ace.setValue(newval, 1)
      } else {
        this.$ace.setValue(this.value, 1)
      }
      // 设置是否为只读模式
      this.$ace.setReadOnly(conf.readOnly)
      //Tab大小
      session.setTabSize(conf.tabSize)
      session.setUseSoftTabs(true)
      // 不显示打印边距
      this.$ace.setShowPrintMargin(false)
      // 自动换行
      session.setUseWrapMode(true)
      // 初始化ace对象
      this.$emit('init', this.$ace)
      // 绑定失去焦点事件回调
      this.$ace.on('blur', ($editor, $fn) => {
        let content = this.$ace.getValue()
        if (this.myConfig.lang === 'json' && this.isJsonString(content)) {
          content = JSON.parse(content)
        }
        this.$emit('input', content, $editor, $fn)
      })
    }
  },
  watch: {
    config: {
      handler(newVal, oldVal) {
        if (newVal === oldVal) return
        this.myConfig = Object.assign({}, defaultConfig, newVal);
        this.initAce(this.myConfig)
      },
      deep: true
    },
    value: {
      handler(newVal, oldVal) {
        if (newVal === oldVal) return
        this.initAce(this.myConfig)
      },
      deep: true
    }
  }
}
</script>
<style scoped>
  .ace-full-screen {
    position: fixed !important;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    width: 100% !important;
    height: 100% !important;
    z-index: 10000;
  }
</style>
