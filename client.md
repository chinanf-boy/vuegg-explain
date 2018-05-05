## client

`vuegg/client`

---


---

## package.json

``` js
  "scripts": {
    "dev": "node build/dev-server.js",
    "start": "node build/dev-server.js",
    "build": "node build/build.js",
```
<details>

``` js
    "unit": "cross-env BABEL_ENV=test karma start test/unit/karma.conf.js --single-run",
    "e2e": "node test/e2e/runner.js",
    "test": "npm run unit && npm run e2e",
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs",
    "clean-svg:prod": "svgo -p 2 -f src/assets/svgs/raw/product/ -o src/assets/svgs/clean/product/",
    "clean-svg:sys": "svgo -p 2 -f src/assets/svgs/raw/system/ -o src/assets/svgs/clean/system/",
    "clean-svg:sys-act": "svgo -p 2 -f src/assets/svgs/raw/system/actions/ -o src/assets/svgs/clean/system/actions/",
    "clean-svg:sys-els": "svgo -p 2 -f src/assets/svgs/raw/system/elements/ -o src/assets/svgs/clean/system/elements/",
    "clean-svg:sys-edit": "svgo -p 2 -f src/assets/svgs/raw/system/editor/ -o src/assets/svgs/clean/system/editor/",
    "clean-svg:all": "npm run clean-svg:prod  && npm run clean-svg:sys && npm run clean-svg:sys-act  && npm run clean-svg:sys-els && npm run clean-svg:sys-edit",
    "svg2icon": "vsvg -s src/assets/svgs/clean -t src/assets/icons",
    "optimicons": "npm run clean-svg:all && npm run svg2icon"
  },
```

</details>

从`package.json`来说, 你很难看出哪里是起点, 各种构建/测试/缩放阿

但是我们又知道

这个项目是由`vue-cli` - `webpack模版`构建的

所以, 尽管从`src`开始就好😊

---


## src

### main.js

> 对于使用`vue-cli`的项目来说

`src/main.js` 就是应用的中心

``` es6
import Vue from 'vue'
import VueSVGIcon from 'vue-svgicon'
// 可以将 .svg => vue component
import VueMDCAdapter from 'vue-mdc-adapter'
// Material 组件 for VueJS
import Tooltip from 'vue-directive-tooltip'
// vue-鼠标停靠-输出信息-工具
import localforage from 'localforage' // 缓存能手

import App from './App' // 主架构
import router from './router/'
import store from './store/'

import './theme.scss'

import 'normalize.css' // 初始化
import 'dialog-polyfill/dialog-polyfill.css'
// dialog plyfill
import 'vue-directive-tooltip/css/index.css'

localforage.config({ name: 'vuegg' })// 定义本地缓存名称

Vue.use(VueSVGIcon)
Vue.use(VueMDCAdapter)
Vue.use(Tooltip, {
  class: 'tooltip-vuegg',
  placement: 'bottom',
  delay: 50
}) // 

Vue.config.productionTip = false
// 生产提示

/* eslint-disable no-new */
const vm = new Vue({
  el: '#app',
  store,
  router,
  template: '<App/>',
  components: { App }
})

export default vm

```

入口, 观全局

> 一般来说, 不会对 css 属性深入 😄

---

### App.vue

#### 窗口限制

``` html
    <div class="viewport-splash_wrapper">
      <div class="viewport-splash_content">
        <svgicon icon="product/vuegg" width="180" height="180" :original="true"></svgicon>
        <h3>Sorry about that!</h3>
        <p>
          It looks like your screen is too small to use vuegg properly,
          please increase your browser size or visit on a larger device.
        </p>
        <p><b>NOTE</b>: As for today, vuegg does not support touch devices.</p>
      </div>
    </div>
```

通过 `class="viewport-splash_wrapper"` 在 css 中 媒体查询

``` css
@media screen and (max-width: 1024px) {
      .viewport-splash_wrapper {
          // ...
```

窗口大小一定要大于 `1024px` 不然, 就甩你一脸提示⚠️

#### svgicon

记得 `package.json` 那些

``` js
"svg2icon": "vsvg -s src/assets/svgs/clean -t src/assets/icons",
// ... 
```

或者可以看[官方->](https://github.com/MMF-FE/vue-svgicon)

或者我简短说说, 

1. 通过命令行将 `svg`格式的图片 转成 可以在 vue 使用的`component`

``` js
// 自动生成的
var icon = require('vue-svgicon')
icon.register({
  'product/vuegg': { // 组件注册-名称
```

因为全局注册了`Vue.use(VueSVGIcon)`

2. `script`-引用, 使用在 `template`

``` js
<script>
import '@/assets/icons/product/vuegg'
```

``` html
<!-- icon 名称 和 其他属性 -->
<svgicon icon="product/vuegg" width="180" height="180" :original="true"></svgicon>
```

#### mixins: [redoundo]

[>>官方](https://cn.vuejs.org/v2/api/#mixins)

就是`先调用 mixins` 的 `再调用 被 mixins` 的

那么 [redoundo](#redoundo) 做了什么

> 制作-可视化-vue-网页, 我们需要回退和前进功能

#### 初始化项目

``` js
  mounted: function () {
    this.initializeState() // 只是一个开始输出信息
    this.loadVueggProject({origin: 'local'})
    // 找啊找啊 , 找项目
  },
```

---

在找之前

我们说过, 所有的全局状态都是由`vuex`行为控制的

而这个时候, vuex 的布置就有所考究了

总体来说分 `结构`与`功能 `

[这里 >>> 有关vuex-布置-结构](./store.conformation.md)

那么下面只讲 功能如何实现

---

##### loadVueggProject

`client/src/store/actions/projectAct.js`

``` js
 [types.loadVueggProject]: async function ({ state, dispatch, commit }, { origin, userName, repoName, content }) {
    commit(types._toggleBlockLoadingStatus, true)

    let project
    switch (origin) {
      case 'local': project = await localforage.getItem('local-checkpoint'); break
      case 'pc': project = content; break
      case 'github':
        // const token = await localforage.getItem('gh-token')
        const owner = userName || state.oauth.authenticatedUser.login
        const repo = repoName || state.project.title.replace(/[^a-zA-Z0-9-_]+/g, '-')

        // let ghFile = await api.getVueggProject(owner, repo, token)
        let ghFile = await api.getVueggProject(owner, repo)

        ghFile
          ? project = ghFile.data.data.content
          : showSnackbar(owner + '/' + repo + ' is not a valid repository')
        break
      default: project = await localforage.getItem('local-checkpoint')
    }

    if (project) {
      store.replaceState(newState(JSON.parse(atob(project))))
      commit(types.addProject)
      if (origin === 'github') localforage.setItem('gh-repo-name', repoName)

      await dispatch(types.checkAuth)
    }
    commit(types._toggleBlockLoadingStatus, false)
  },
```
---

#### redoundo

> 制作-可视化-vue-网页, 我们需要回退和前进功能

1. 增加 `done` 和 `undone` 一进一退的存储

2. 验证模式 `vuex` 对「不是`_`开头行为」「最大`250`行为存储」

3. `vuex` 与 前端按钮 联系

所有的全局状态变化都是通过 `vuex` 的行为驱动的

- 3.1 增加事件触发「主要是给前端按钮 触发 `vuex`行为」

- 3.2 对一进一退的存储 的增减

- 3.3 以及按钮的可用性判断



<details>

``` js
const redoundo = {
  data: function () { // <===== 1
    return {
      done: [],
      undone: []
    }
  },

  created: function () {
    this.$store.subscribe((mutation, state) => {
// <====== 2

      // If the history size is reached, the eldest state will be removed
      if (this.done.length === MAX_HISTORY) this.done.shift()

      // It won't save the state of mutations leaded by '_'
      if (mutation.type.charAt(0) !== '_') {
        this.done.push(cloneDeep(state))
        this.undone = []

        // To display if changes had happened to the project
        this.updateCanRedoUndo()
        this.$store.dispatch(checkLastSaved)
      }
    })

// <===== 3 添加
    this.$root.$on('undo', this.undo)
    this.$root.$on('redo', this.redo)
  },
// <===== 3 清除

  beforeDestroy: function () {
    this.$root.$off('undo', this.undo)
    this.$root.$off('redo', this.redo)
  },

  computed: {
    canUndo () {
    // There should always be at least one state (initializeState)
      return this.done.length > 1
    },
    canRedo () {
      return this.undone.length > 0
    }
  },

  methods: {
      // 3.1
    undo () {
      if (this.canUndo) {
          // 3.2
        this.undone.push(this.done.pop())
        let undoState = this.done[this.done.length - 1]
        this.$store.replaceState(cloneDeep(undoState))
        this.$root.$emit('rebaseState')
        this.updateCanRedoUndo()
      }
    },
      // 3.1

    redo () {
      if (this.canRedo) {
          // 3.2
          
        let redoState = this.undone.pop()
        this.done.push(redoState)
        this.$store.replaceState(cloneDeep(redoState))
        this.$root.$emit('rebaseState')
        this.updateCanRedoUndo()
      }
    },

    // 3.3
    updateCanRedoUndo () {
      this.$store.commit(_toggleCanUndo, this.canUndo)
      this.$store.commit(_toggleCanRedo, this.canRedo)
    }
  }
}
```
</details>