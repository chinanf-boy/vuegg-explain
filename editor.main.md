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

- [ ] [mr-container explain](./mr-container.md)

---

第一步
- [x] [`@selectstop="selectStopHandler"`](#selectstop)  括起来一/多个元素

需要选择
- [ ] [`@arrows="arrowsHandler"`](#arrows)  箭头移动
- [ ] [`@moving="movingHandler"`](#moving)  移动样式变化
- [ ] [`@movestop="moveStopHandler"`](#movestop)  移动停止, 样式变化
- [ ] [`@resizestop="resizeStopHandler"`](#resizestop)  大小变化
- [ ] [`@clearselection="clearSelectionHandler"`](#clearselection)  清空选择
- [ ] [`@delete="deleteHandler"`](#delete)  删除
- [ ] [`@copy="copyHandler"`](#copy)  复制
- [ ] [`@cut="cutHandler"`](#cut)  剪切
- [ ] [`@paste="pasteHandler"`](#paste)  粘贴

已经按住了
- [x] [`@drop="dropHandler"`](#drop)  放下 

不需要选择
- [ ] [`@undo="$root.$emit('undo')"`](#undo) 回退
- [ ] [`@redo="$root.$emit('redo')"`](#redo) 复原

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

### Moving

>

<details>

``` js

```

</details>

---

### Movestop

>

<details>

``` js

```

</details>

---

### Resizestop

>

<details>

``` js

```

</details>

---

### Selectstop

>

<details>

``` js

```

</details>

---

### Clearselection

>

<details>

``` js

```

</details>

---

### Delete

>

<details>

``` js

```

</details>

---

### Copy

>

<details>

``` js

```

</details>

---

### Cut

>

<details>

``` js

```

</details>

---

### Paste

>

<details>

``` js

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

---

``` js
  computed: {
    isActive () {
      return (this.selectedElements.findIndex(el => el.id === this.elem.id) !== -1)
    },

    componentRef () {
      return this.projectComponents[this.projectComponents.findIndex(comp => comp.name === this.elem.name)]
    },

    ...mapState({
      selectedElements: state => state.app.selectedElements,
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
