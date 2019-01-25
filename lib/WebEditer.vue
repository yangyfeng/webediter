<template>
  <div class="ace-container" :style="{width: width + 'px', height: height + 'px'}">
    <div class="ace-editer" ref="ace-editer">{{value}}</div>
  </div>
</template>
<script>
// 主题 chrome clouds cclipse gitHub tomorrow monokai terminal twilight dawn dreamweaver
// 模式 css ejs  html jade java json javascript json
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
      default: 500
    },
    value: {
      type: [Object, Array, String],
      default () {
        return {}
      }
    },
    theme: {
      type: String,
      default: 'monokai'
    },
    mode: {
      type: String,
      default: 'json'
    },
    readonly: {
      type: Boolean,
      default: false
    },
    format: {
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
      editer.setFontSize(14)
      // 自动换行,设置为off关闭
      editer.setOption('wrap', 'free')
      // 设置值 arg1 字符类型 arg2 在哪里设置新值。 undefined或0是selectAll，-1是文档开头，1是结尾
      // editer.setValue()
      editer.on('blur', (e) => {
        let newVal = editer.getValue()
        // 是不是json模式
        if (this.mode !== 'json') {
          return this.$emit('input', newVal)
        }
        if (this.isjson(newVal)) {
          if (this.format) editer.setValue(this.formatJson(newVal))
          this.$emit('input', JSON.parse(newVal))
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
    },
    formatJson (json, options) {
      let reg = null
      let formatted = ''
      let pad = 0
      let PADDING = '  '
      options = options || {}
      options.spaceAfterColon = !options.spaceAfterColon || true
      if (typeof json !== 'string') {
        json = JSON.stringify(json)
      } else {
        json = JSON.parse(json)
        json = JSON.stringify(json)
      }
      /* eslint-disable */
      reg = /([\{\}])/g
      json = json.replace(reg, '\r\n$1\r\n')
      reg = /([\[\]])/g
      json = json.replace(reg, '\r\n$1\r\n')
      reg = /(\,)/g
      json = json.replace(reg, '$1\r\n')
      reg = /(\r\n\r\n)/g
      json = json.replace(reg, '\r\n')
      reg = /\r\n\,/g
      json = json.replace(reg, ',')
      if (!options.newlineAfterColonIfBeforeBraceOrBracket) {
        reg = /\:\r\n\{/g
        json = json.replace(reg, ':{')
        reg = /\:\r\n\[/g
        json = json.replace(reg, ':[')
      }
      if (options.spaceAfterColon) {
        reg = /\:/g
        json = json.replace(reg, ':')
      }
      json.split('\r\n').forEach((node, index) => {
        let i = 0
        let indent = 0
        let padding = ''
        if (node.match(/\{$/) || node.match(/\[$/)) {
          indent = 1
        } else if (node.match(/\}/) || node.match(/\]/)) {
          if (pad !== 0) {
            pad -= 1
          }
        } else {
          indent = 0
        }
        for (i = 0; i < pad; i++) {
          padding += PADDING
        }
        formatted += padding + node + '\r\n'
        pad += indent
      })
      return formatted
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
