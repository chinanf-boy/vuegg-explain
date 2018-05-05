## store

`store` 是 vuex 使用的昵称

---

https://vuex.vuejs.org/zh-cn/
https://github.com/vuejs/vuex

---

`client/src/store`

### vuex

1. `state`

`store/state.js`

数据存储

2. `mutaioins`

`store/mutations/*.js`

同步: 对数据进行同步操作, 加减之类的啊

3. `actions`

`store/actions/*.js`

异步: 都是需要对远端Api, 获取数据后, 使用`mutaioins-同步`操作数据

> actions 包住 mutaioins, 或 actions 包住 actions

4. `getters`

`store/getters/*.js`

过滤: 快速获取数据中的某个值, 而存在的, 比如`较为深的对象`中的`某个值返回`

vuex 主要由这 4 部分 组成

> 那么 `store/types.js` 又是什么

5. `types`

不管是同步/异步/过滤行为, 不过是有名称的函数运行, 作者把所有的行为函数-名称集合

> 行为集合

---

### 最后

`store/index.js`

 一网打尽

``` js
import Vue from 'vue'
import Vuex from 'vuex'

import state from './state'
import getters from './getters'
import actions from './actions'
import mutations from './mutations'

Vue.use(Vuex)

//在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。
export default new Vuex.Store({
  strict: process.env.NODE_ENV !== 'production', // <=== 严格？
  state,
  getters,
  actions,
  mutations
})
```

## 其实还没有结束😊

### 分布

``` js
'./getters'
'./actions'
'./mutations'
```

作者在各自的文件夹中是这样分布的

``` js
store
    - actions
        - authAct.js        // auth 验证 范畴
        - elementAct.js     // element 元素 范畴
        - index.js
        - pageAct.js        // page 页面 范畴
        - projectAct.js     // project 项目 范畴
    - getters
        - componentGet.js   // component 组件 范畴
        - elementGet.js
        - index.js
        - pageGet.js
    - mutations
        - appMut.js         // app 应用 范畴
        - authMut.js
        - componentMut.js
        - elementMut.js
        - index.js
        - pageMut.js
        - projectMut.js
```

全局存储的重要性, 不言而喻, 正确对待 `vuex`的结构

是持续发展的前提