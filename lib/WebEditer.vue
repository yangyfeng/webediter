<template>
  <div class="ace-container" :style="{width: width + 'px', height: height + 'px'}">
    <div class="ace-editer" ref="ace-editer">{{value}}</div>
  </div>
</template>
<script>
import Ace from 'ace-builds'
// 在 webpack 环境中使用必须要导入
import 'ace-builds/webpack-resolver'
export default {
  props: {
    width: {
      type: Number,
      default: 400
    },
    height: {
      type: Number,
      default: 600
    },
    value: {
      type: [Object, Array, String, Number, Boolean],
      default: ''
    },
    theme: {
      type: String,
      default: 'idle_fingers'
    },
    mode: {
      type: String,
      default: 'txt'
    },
    readonly: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      editer: null
    }
  },
  mounted () {
    this.init()
  },
  beforeDestroy () {
    this.editer.destroy()
    this.editer = null
  },
  created () {
    require('ace-builds/src-min-noconflict/theme-' + this.theme)
  },
  methods: {
    init () {
      // 获取挂载实例元素
      let aceDom = this.$refs['ace-editer']
      if (!aceDom) return
      // 初始化实例
      this.editer = Ace.edit(aceDom)
      let editer = this.editer
      // 设置语言
      editer.session.setMode(`ace/mode/${this.mode}`)
      // 设置主题
      editer.setTheme(`ace/theme/${this.theme}`)
      // 设置只读
      editer.setReadOnly(this.readonly)
      // 设置高亮
      editer.setHighlightActiveLine(true)
      // 字体大小
      editer.setFontSize(12)
      // 自动换行,设置为off关闭
      editer.setOption('wrap', true)
      // 设置值 arg1 字符类型 arg2 在哪里设置新值。 undefined或0是selectAll，-1是文档开头，1是结尾
      editer.on('blur', (e) => {
        if (this.readonly) return
        let newVal = editer.getValue()
        // 是不是json模式
        if (this.mode !== 'json') {
          this.$emit('input', newVal)
          this.$emit('onChange', newVal)
          return false
        }
        if (this.isjson(newVal)) {
          this.$emit('input', JSON.parse(newVal))
          this.$emit('onChange', JSON.parse(newVal))
        }
      })
    },
    isjson (str) {
      if (typeof str !== 'string') return
      let is = false
      try {
        JSON.parse(str)
        is = true
      } catch (error) {
        console.info('不是json')
      }
      return is
    }
  }
}
</script>
<style scoped>
  .ace-editer {
    width: 100%;
    height: 100%;
    overflow: auto;
  }
</style>
