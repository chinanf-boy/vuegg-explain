## main

画布

- [ ] [index](#index)
- [ ] [Stage](#stage)
- [ ] [StageEl](#stageel)

---

其实, 前端的所有操作, 都是用户完成的, 当然我们可以初始化界面的表现

但, 随后的变化, 是跟随用户的脚步👣走的

所以我认为, 只要函数命名准确, 单从 `模版-template|html` 的角度

我们就能纵观数据的大概流向

<details>

说到底不就是

全局缓存「vuex」+ 本地缓存」+ 局部缓存「单个Vux」+ 远程缓存「Api」 = 数据 

显示 + 隐藏 .etc = 视图 

同步 + 异步 + 过滤 + Api请求「甚至可以归类成异步」= 行为 

所以才叫 `MvvC` - `Vue`

</details>

### index

``` html
<template>
  <div class="mainegg">
    <stage v-if="selectedPage" :page="selectedPage"></stage>
  </div>
</template>
```

`selectedPage` 换页面的状态值

---

在开始进入之前

我们过一下

`index` -> `stage` -> `mr-container` -> for `page.children` {
  `stage-el`  <= `mr-el -> element` <- 有时stage-el-孩子
}



---

### Stage

<details>

``` html
<!-- selectedElements 每个页面下的活动元素 -->
  <mr-container
    :id="page.id"
    :style="pageStyles" 
    :class="[page.classes, {stage: true}]"
    :activeElements="selectedElements"
    @arrows="arrowsHandler"
    @moving="movingHandler"
    @movestop="moveStopHandler"
    @resizestop="resizeStopHandler"
    @selectstop="selectStopHandler"
    @clearselection="clearSelectionHandler"
    @delete="deleteHandler"
    @copy="copyHandler"
    @cut="cutHandler"
    @paste="pasteHandler"
    @drop="dropHandler"
    @undo="$root.$emit('undo')"
    @redo="$root.$emit('redo')"
  >

```

### children-use

`children` 被 `stage-el` 分了

``` js
    <stage-el
      v-for="element in page.children"
      :key="element.id"
      :elem="element">
    </stage-el>

  </mr-container>
```

这里各种@操作


---

在这之前, 我们看到 `mr-container` 组件

主要是把 `快捷键`与`对应的@***操作`联系起来

而且, 所有的操作都是通过, `mr-container` 先的

- [x] [mr-container explain](./mr-container.md) <==== 先看这里

同时也是, 点按拖动, 放下的起点

---

第一步
- [x] [`@selectstop="selectStopHandler"`](#selectstop)  括起来一/多个元素

需要选择
- [x] [`@arrows="arrowsHandler"`](#arrows)  箭头移动
- [x] [`@moving="movingHandler"`](#moving)  移动样式变化
- [x] [`@movestop="moveStopHandler"`](#movestop)  移动停止, 样式变化
- [x] [`@resizestop="resizeStopHandler"`](#resizestop)  大小变化
- [x] [`@clearselection="clearSelectionHandler"`](#clearselection)  清空选择
- [x] [`@delete="deleteHandler"`](#delete)  删除
- [x] [`@copy="copyHandler"`](#copy)  复制
- [x] [`@cut="cutHandler"`](#cut)  剪切
- [x] [`@paste="pasteHandler"`](#paste)  粘贴

已经按住了
- [x] [`@drop="dropHandler"`](#drop)  放下 

不需要选择
- [x] [`@undo="$root.$emit('undo')"`](#undo) 回退
- [x] [`@redo="$root.$emit('redo')"`](#redo) 复原

[>> $emit('undo')+$emit('redo')](./readme.md#redoundo)

---

### selectstop

> 有许多操作, 都需要知道操作的对象是哪个, 这里只是 括起来的操作

> 当然还有 

- [x] [直接点 元素的操作](#activated)

<details>

``` js
    selectStopHandler (selectionBox) {
      if ((selectionBox.top === selectionBox.bottom && selectionBox.left === selectionBox.right) ||
          (this.page.children.length === 0)) return
    // 页面中要有 元素, 还要 有选择痕迹

// 然后自然就是 计算 元素是否在这个范围
      let selectedElements = []
      this.page.children.forEach(childEl => {
        const child = (childEl.global) ? {...childEl, ...this.getComponentRef(childEl), id: childEl.id} : childEl

        let childTop = getComputedProp('top', child)
        let childLeft = getComputedProp('left', child)
        let childBottom = getComputedProp('height', child, this.page) + childTop
        let childRight = getComputedProp('width', child, this.page) + childLeft

        if (((childTop <= selectionBox.bottom) && (childLeft <= selectionBox.right) &&
            (childBottom >= selectionBox.top) && (childRight >= selectionBox.left)) ||
            ((childTop <= selectionBox.bottom) && (childRight >= selectionBox.left) &&
            (childBottom >= selectionBox.top) && (childLeft <= selectionBox.right))) {
          selectedElements.push(child)
        }
      })
// 添加 全局存储
      if (selectedElements.length > 0) {
        this._addSelectedElements(selectedElements) 
// 注意是 _addSelectedElements s s s
      }
    },
```

</details>

---


### arrows

> 在选择了元素 , 通过箭头按键移动

<details>

``` js
    arrowsHandler ({direction, shiftKey}) {
      if (this.selectedElements.length > 0) { // 有选择
        let diff = shiftKey ? 10 : 1

        let addedTop = 0
        let addedBottom = 0
        let addedLeft = 0
        let addedRight = 0

        switch (direction) {
          case 'up': addedTop -= diff; addedBottom += diff; addedLeft = addedRight = null; break
          case 'down': addedBottom -= diff; addedTop += diff; addedLeft = addedRight = null; break
          case 'left': addedLeft -= diff; addedRight += diff; addedTop = addedBottom = null; break
          case 'right': addedRight -= diff; addedLeft += diff; addedTop = addedBottom = null; break
        }

        this.selectedElements.map(el => {
          let compTop = getComputedProp('top', el)
          let compBottom = getComputedProp('bottom', el)
          let compLeft = getComputedProp('left', el)
          let compRight = getComputedProp('right', el)

          let top = (addedTop && ((compTop + addedTop) >= 0) && ((compBottom + addedBottom) >= 0))
            ? (compTop + addedTop) : null
          let bottom = (addedBottom && ((compBottom + addedBottom) >= 0) && ((compTop + addedTop) >= 0))
            ? (compBottom + addedBottom) : null
          let left = (addedLeft && ((compLeft + addedLeft) >= 0) && ((compRight + addedRight) >= 0))
            ? (compLeft + addedLeft) : null
          let right = (addedRight && ((compRight + addedRight) >= 0) && ((compLeft + addedLeft) >= 0))
            ? (compRight + addedRight) : null

          if (top || bottom || left || right) {
            this.moveElement({ elId: el.id, pageId: this.page.id, top, bottom, left, right })
          }
        })
        this.rebaseSelectedElements() // 每个选择元素 重新装载 全局存储
      }
    },
```
</details>

---

### Moving

> 移动时候, 是否容器的计算 是一个鼠标样式变化

<details>

``` js
    movingHandler (absMouseX, absMouseY) {
      this.dropContainer = this.getContaineggOnPoint(absMouseX, absMouseY)
// 1. 鼠标位置所有的元素
// 2. 确定是 容器 且 不是组件 且 不是父 且 不是自己/子


      this.toggleDroppableCursor(!!this.dropContainer) // 是否能添加到容器 鼠标显示➕
    },
```

</details>

---

### Movestop

> 移动停下后， 计算是否 脱离父辈

<details>

``` js
    moveStopHandler (moveStopData) {
      const containegg = this.getContaineggOnPoint(moveStopData.absMouseX, moveStopData.absMouseY)
      
      const parentId = containegg ? containegg.id : null
// parentId 如果有 证明要 跳出来
      moveStopData.moveElData.map(moveData => this.moveElement({
// moveElement 全局 action
// 1. -改变元素父辈
// 2. 改变 鼠标与原位置的计算位置
        ...moveData,
        pageId: this.page.id,
        parentId,
        mouseX: moveStopData.relMouseX,
        mouseY: moveStopData.relMouseY
      }))

      this.rebaseSelectedElements()
      this.toggleDroppableCursor(false)
      this.dropContainer = null
    },
```

</details>

---

### Resizestop

> 缩放 停止后, 前端样式, 已经变化， 我们还要把 全局存储改变

<details>

``` js
    resizeStopHandler (resStopData) {
      resStopData.map(resElData => this.resizeElement({...resElData, pageId: this.page.id}))
// resizeElement
// 对每个元素 更新
      this.rebaseSelectedElements()
    },
```

</details>

---

### Clearselection

> 清空 全局选择元素

<details>

`mr-container` 运行 `$emit('clearselection')`

`@clearselection="clearSelectionHandler"`

``` js
    clearSelectionHandler () {
      if (this.selectedElements.length > 0) this._clearSelectedElements()
    },
```

</details>

---

### Delete

> 删除

<details>

``` js
    deleteHandler () {
      if (this.selectedElements.length > 0) {
        this.selectedElements.map(el => this.removeElement({page: this.page, elId: el.id}))
      }
    },
```

</details>

---

### Copy

> 组件缓存 clipboard 先放好

<details>

``` js
    copyHandler () {
      if (this.selectedElements.length > 0) {
        this.clipboard = []
        this.selectedElements.map(el => this.clipboard.push(cloneDeep(el)))
      }
    },
```

</details>

---

### Cut

> 先放好, 再移除

<details>

``` js
    cutHandler () {
      if (this.selectedElements.length > 0) {
        this.clipboard = []
        this.selectedElements.map(el => {
          this.clipboard.push(cloneDeep(el))
          this.removeElement({page: this.page, elId: el.id})
        })
      }
    },
```

</details>

---

### Paste

> 将缓存, 放入全局

<details>

``` js
    pasteHandler () {
      if (this.clipboard.length > 0) {
        this.clipboard.map(el => {
          this.registerElement({pageId: this.page.id, el, global: el.global})
        })
      }
    },
```

</details>

---

### Drop

[要按住-dragstart 来自画笔](./editor.drawegg.md#dragstart)

[才能放下-drop](#drop)

<details>

``` html
    @drop="dropHandler"
```

``` js
    dropHandler (e) {
      const mainContainer = document.getElementById('main')
      let element = JSON.parse(e.dataTransfer.getData('text/plain'))
// 我们拿到了, 从 元素栏 那里 setData
      let height = getComputedProp('height', element, this.page)
      let width = getComputedProp('width', element, this.page)
      let top = e.pageY + mainContainer.scrollTop - mainContainer.offsetTop - this.$el.offsetTop - (height / 2)
      let left = e.pageX + mainContainer.scrollLeft - mainContainer.offsetLeft - this.$el.offsetLeft - (width / 2)
// 计算四边距离
      const fixedElement = fixElementToParentBounds({top, left, height, width}, this.page)
      element = {...element, ...fixedElement} // 统计

      this.registerElement({pageId: this.page.id, el: element, global: e.shiftKey})
// 再一次扔给, 所在页面的 children 列表
    },
```

如果你回到 元素栏 「点击和按住移动」, 

- 区别:也就是 四边距离

- 相同: `registerElement` -> 建立的元素 放入所在页面 的 `children` 列表

那么然后呢, `children` 列表 在哪里用到了[ 使用`children` 列表 >> ](#children-use)

</details>

---


### Undo

>

<details>

``` js

```

</details>

### Redo

>

<details>

``` js

```

</details>



</details>

### StageEl

<details>

``` js
import { mapState, mapMutations } from 'vuex'
import { _clearSelectedElements, _addSelectedElement } from '@/store/types'

import MrEl from '@/components/editor/common/mr-vue/MrEl'
import StageEl from './StageEl'

export default {
  name: 'stage-el',
  props: ['elem', 'isPlain'],
  components: { MrEl },
  render: function (createElement) {
    let elementO = (this.elem.global) ? {...this.elem, ...this.componentRef, id: this.elem.id} : this.elem

    let styles = elementO.styles
    if (this.isPlain && elementO.egglement) {
      styles = {
        ...elementO.styles,
        position: 'absolute',
        zIndex: elementO.zIndex,
        minWidth: elementO.minWidth,
        minHeight: elementO.minHeight,
        top: (typeof elementO.top === 'number') ? (elementO.top + 'px') : elementO.top,
        left: (typeof elementO.left === 'number') ? (elementO.left + 'px') : elementO.left,
        bottom: (typeof elementO.bottom === 'number') ? (elementO.bottom + 'px') : elementO.bottom,
        right: (typeof elementO.right === 'number') ? (elementO.right + 'px') : elementO.right,
        width: (typeof elementO.width === 'number') ? (elementO.width + 'px') : elementO.width,
        height: (typeof elementO.height === 'number') ? (elementO.height + 'px') : elementO.height
      }
    }

    const data = {
      'class': elementO.classes,
      'style': styles,
      'attrs': {
        id: elementO.id,
        'data-global': elementO.global,
        'data-egglement': elementO.egglement,
        'data-containegg': elementO.containegg,
        'data-componegg': elementO.componegg,
        'data-wrappegg': elementO.wrappegg,
        ...elementO.attrs
      }
    }

    let children = []
    if (elementO.text) {
      children.push(elementO.text)
    } else if (elementO.children) {
      for (let child of elementO.children) {
        children.push(createElement(StageEl, {
          'props': {
            elem: child,
            isPlain: elementO.componegg || elementO.wrappegg || this.isPlain
          }
        }))
      }
    }

    let stageElem
    if (this.isPlain) {
      stageElem = createElement(elementO.type, data, children)
    } else {
      let mrElProps = {
        active: this.isActive,
        left: elementO.left,
        top: elementO.top,
        right: elementO.right,
        bottom: elementO.bottom,
        zIndex: elementO.zIndex,
        width: elementO.width,
        height: elementO.height,
        minWidth: elementO.minWidth,
        minHeight: elementO.minHeight
      }

      stageElem = createElement(MrEl, {
        'props': (elementO.global) ? {...mrElProps, handles: null} : mrElProps,
        'on': { activated: this.activatedHandler }
      }, [ createElement(elementO.type, data, children) ])
    }

    return stageElem
  },
```

### getContaineggOnPoint
### toggleDroppableCursor

``` js
    getContaineggOnPoint (x, y) {
      const movingEggs = this.selectedElements
      const parentsIds = movingEggs.map(egg => egg.id.substring(0, egg.id.lastIndexOf('.')))
      const commonParentId = parentsIds.every((val, i, arr) => val === arr[0]) ? parentsIds[0] : null
      const elementsOnPoint = elementsFromPoint(x, y)

      for (let el of elementsOnPoint) {
        if (el.id === commonParentId) return null
        if ((el.dataset.mrContainer) ||
          (
            (el.dataset.containegg) &&
            (!el.dataset.componegg) &&
            (movingEggs.every(egg => !el.id.includes(egg.id)))
          )
        ) return el
      }
      return null
    },

    toggleDroppableCursor (isDroppable) {
      isDroppable
        ? document.documentElement.classList.add('droppable')
        : document.documentElement.classList.remove('droppable')
    },
```

### activated

其实又是一个 `on-emit`

- `'on': { activated: this.activatedHandler }`
- `on` -> `MrEl` {`$emit`}

``` js
// MrEl.vue
    @mousedown="e => $emit('activated', e)"
    @mousedown.meta.capture="e => $emit('activated', e)"
    @mousedown.ctrl.capture="e => $emit('activated', e)"
```
```js
// StageEl.vue
        'on': { activated: this.activatedHandler }

```
``` js
  computed: {
    isActive () {
      return (this.selectedElements.findIndex(el => el.id === this.elem.id) !== -1)
    },

    componentRef () {
      return this.projectComponents[this.projectComponents.findIndex(comp => comp.name === this.elem.name)]
    },

    ...mapState({
      selectedElements: state => state.app.selectedElements, // 点击元素这个会变
      projectComponents: state => state.project.components
    })
  },
  methods: {
    activatedHandler (e) {
      e.stopPropagation()
      e.preventDefault()

// 按住 shift 键 是 再加一个元素被选择
      if (e.shiftKey && !this.isActive) {
        this._addSelectedElement(this.elem)
// 注意是 _addSelectedElement
// no s
      } else if (!e.shiftKey && !this.isActive) {
// 不按住 shift 是， 只有一个元素被选择
        this._clearSelectedElements()
        this._addSelectedElement(this.elem)
      }
    },

    ...mapMutations([_clearSelectedElements, _addSelectedElement])
  }
}

```


</details>
