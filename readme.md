## webediter

### 安装 
```
npm install webediter --save
```
### 使用
```
// 导入
import WebEditer from 'webediter'

// 组件
<WebEditer v-model="value" :width="1000" mode="json" :format="true" ></WebEditer>

// 主题 chrome clouds cclipse gitHub tomorrow monokai terminal twilight dawn dreamweaver

// 模式 css ejs  html jade java json javascript json

// 属性
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
}