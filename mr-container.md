## mr-container

快捷键和点选操作

`editor/common/mr-vue/MrContainer.vue`

---

1. [x] [on/emit](#on-emit)
2. [x] [鼠标点击](#mouseDown)

> 三种方式都是从[0. 鼠标点击](#mouseDown)开始的

- 选择

        
    1. [this.$emit('clearselection')](./editor.main.md#clearselection) [renderSelectionArea](#renderselectionarea)
    2. 注册 [mousemove](#mousemovehandler) - [mouseup](#mouseuphandler)
    3. `this.$emit('selectstop', this.selectStopData())` - [selectstop](./editor.main.md#selectstop) [selectStopData](#selectstopdata)
    4. 初始化 组件数据
    5. 删除 `mousemove` `mouseup`

- 移动

    1. [this.getParentMr(e.target)](#getparentmr)
    2. 注册 [mousemove](#mousemovehandler) - [mouseup](#mouseuphandler)
    3. `this.mrElements.map(mrEl => this.moveElementBy(mrEl, offX, offY))` - [this.moveElementBy](#moveelementby)
    4. `this.$emit('moving', this.currentAbsPos.x, this.currentAbsPos.y)` - [moving](./editor.main.md#moving) 
    5. `this.$emit('movestop', this.moveStopData())` - [movestop](./editor.main.md#movestop) - [moveStopData](#movestopdata)
    6. 初始化 组件数据    
    7. 删除 `mousemove` `mouseup`

- 缩放

   1. `e.target.classList[1]` [哪个缩放角选择](./mrel.md#handles)
   2. 注册 [mousemove](#mousemovehandler) - [mouseup](#mouseuphandler)
   3. `this.mrElements.map(mrEl => {
          if (!mrEl.children[0].dataset.global) this.resizeElementBy(mrEl, offX, offY)
        })` - [resizeElementBy](#resizeelementby)
   4. `this.$emit('resizestop', this.resizeStopData())`  -  [resizestop](./editor.main.md#resizestop) - [resizeStopData](#resizestopdata)
   5. 初始化 组件数据    
   6. 删除 `mousemove` `mouseup`

#### 总结

1⃣️ 看用户`点啥`
2⃣️ 按住期间, 鼠标位置变化, `视图`先更新
3⃣️ 点按结束, 我们通过`$emit`把`全局存储`更新
剩下的就是快捷键

<details>


- [x] [mouseDown](#mousedown)
- [x] [mouseUpHandler](#mouseuphandler)
- [x] [mouseMoveHandler](#mousemovehandler)
- [x] [renderSelectionArea](#renderselectionarea)
- [x] [resizeElementBy](#resizeelementby)
- [x] [moveElementBy](#moveelementby)
- [x] [fixPosition](#fixposition)
- [x] [moveStopData](#movestopdata)
- [x] [resizeStopData](#resizestopdata)
- [x] [selectStopData](#selectstopdata)

</details>

---

``` html
<template>
  <div data-mr-container="true"
    class="mr-container"
    tabindex="0"
    @mousedown.capture="mouseDownHandler"
    @keydown.esc.stop.prevent="$emit('clearselection')"
    @keydown.delete.exact.stop.prevent="$emit('delete')"
    @keydown.ctrl.67.exact.stop.prevent="$emit('copy')"
    @keydown.meta.67.exact.stop.prevent="$emit('copy')"
    @keydown.ctrl.88.exact.stop.prevent="$emit('cut')"
    @keydown.meta.88.exact.stop.prevent="$emit('cut')"
    @keydown.ctrl.86.exact.stop.prevent="$emit('paste')"
    @keydown.meta.86.exact.stop.prevent="$emit('paste')"
    @keydown.ctrl.90.exact.stop.prevent="$emit('undo')"
    @keydown.meta.90.exact.stop.prevent="$emit('undo')"
    @keydown.ctrl.shift.90.exact.stop.prevent="$emit('redo')"
    @keydown.meta.shift.90.exact.stop.prevent="$emit('redo')"
    @keydown.up.stop.prevent="e => $emit('arrows', {direction: 'up', shiftKey: e.shiftKey})"
    @keydown.down.stop.prevent="e => $emit('arrows', {direction: 'down', shiftKey: e.shiftKey})"
    @keydown.left.stop.prevent="e => $emit('arrows', {direction: 'left', shiftKey: e.shiftKey})"
    @keydown.right.stop.prevent="e => $emit('arrows', {direction: 'right', shiftKey: e.shiftKey})"
    @drop.prevent="e => $emit('drop', e)"
    @dragover.prevent
  >
    <slot></slot> 
    <!-- 插槽 -->
    <div ref="selectionArea" v-show="selecting" class="selection-area"></div>
    <!-- 记录 划过的矩形 -->
  </div>
</template>

```

### on-emit

`@ - $on` - `$emit`

对应每一个 组件来说 像

``` html
<button @click="clickRun" type="text" />
```

其实是 `button.$on('click', clickRun)` 这样 的 定义

然后, `原生 DOM click `触发的函数中 -包含- `button.$emit('click')` 也被触发了

正如那一堆 `@ - $on` 的定义

``` js
// editor/main/Stage.vue
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
```

才能被 `$emit` 使用

``` js
// editor/common/mr-vue/MrContainer.vue
    @keydown.esc.stop.prevent="$emit('clearselection')"
    @keydown.delete.exact.stop.prevent="$emit('delete')"
    @keydown.ctrl.67.exact.stop.prevent="$emit('copy')"
    @keydown.meta.67.exact.stop.prevent="$emit('copy')"
    @keydown.ctrl.88.exact.stop.prevent="$emit('cut')"
    @keydown.meta.88.exact.stop.prevent="$emit('cut')"
    @keydown.ctrl.86.exact.stop.prevent="$emit('paste')"
    @keydown.meta.86.exact.stop.prevent="$emit('paste')"
    @keydown.ctrl.90.exact.stop.prevent="$emit('undo')"
    @keydown.meta.90.exact.stop.prevent="$emit('undo')"
    @keydown.ctrl.shift.90.exact.stop.prevent="$emit('redo')"
    @keydown.meta.shift.90.exact.stop.prevent="$emit('redo')"
```

---

### 初始化

``` js
export default {
  name: 'mr-container',
  props: ['activeElements'], // 选择了的元素
  data: function () {
    return {
      initialAbsPos: {x: 0, y: 0},
      initialRelPos: {x: 0, y: 0},
      currentAbsPos: {x: 0, y: 0},
      currentRelPos: {x: 0, y: 0},
      selecting: false,
      moving: false,
      resizing: false,
      handle: null
    }
  },
  computed: {
    mrElements () {
      return this.activeElements.map(el => document.getElementById(el.id).parentElement)
    }
  },
  methods: {

```

### mouseDown

每一次在在画布的点击都来到这里

每一次按下都是一次重新计算

``` js
    mouseDownHandler (e) { // 按下
      let isMrs = false
      this.initialAbsPos = this.currentAbsPos = this.getMouseAbsPoint(e)
      this.initialRelPos = this.currentRelPos = this.getMouseRelPoint(e)

      if (e.target.dataset.mrContainer) { // 选择了画布
        this.$emit('clearselection')
        this.renderSelectionArea({x: -1, y: -1}, {x: -1, y: -1})
        isMrs = this.selecting = true
      } else if (e.target.dataset.mrHandle) { // 选择了 元素 的 缩放角
        isMrs = this.resizing = true
        this.handle = e.target.classList[1] // 哪个缩放角
        // this.$emit('resizestart')
      } else if (this.getParentMr(e.target)) { // 如果有 父辈 除了 画布
        isMrs = this.moving = true
        // this.$emit('movestart')
      }

      if (isMrs) {
        document.documentElement.addEventListener('mousemove', this.mouseMoveHandler, true)
        document.documentElement.addEventListener('mouseup', this.mouseUpHandler, true)
      }
    },

```

### mouseUpHandler

放下

``` js
    mouseUpHandler (e) { // 放开
      // 在对焦之前保存滚动位置并将其重新设置
      const mainContainer = document.getElementById('main')
      let currentScroll = mainContainer.scrollTop
      this.$el.focus()
      mainContainer.scrollTop = currentScroll

      if (this.initialAbsPos !== this.currentAbsPos) {
        if (this.resizing) this.$emit('resizestop', this.resizeStopData())
        else if (this.moving) this.$emit('movestop', this.moveStopData())
        else if (this.selecting) this.$emit('selectstop', this.selectStopData())
        
        // selectstop 会让 在 选择矩形内的 元素 进入全局存储
      }

      this.moving = false
      this.resizing = false
      this.selecting = false
      this.handle = null

      document.documentElement.removeEventListener('mousemove', this.mouseMoveHandler, true)
      document.documentElement.removeEventListener('mouseup', this.mouseUpHandler, true)
    },

```

### mouseMoveHandler

按住移动时

``` js
    mouseMoveHandler (e) {
      const lastAbsX = this.currentAbsPos.x
      const lastAbsY = this.currentAbsPos.y

      this.currentAbsPos = this.getMouseAbsPoint(e)
      this.currentRelPos = this.getMouseRelPoint(e)

      let offX = this.currentAbsPos.x - lastAbsX
      let offY = this.currentAbsPos.y - lastAbsY

      if (this.resizing) { // 重新计算
        this.mrElements.map(mrEl => {
          if (!mrEl.children[0].dataset.global) this.resizeElementBy(mrEl, offX, offY)
        })
        // this.$emit('resizing')
      } else if (this.moving) { // 
        this.mrElements.map(mrEl => this.moveElementBy(mrEl, offX, offY))
        // 还要计算一下 要脱离父辈吗
        this.$emit('moving', this.currentAbsPos.x, this.currentAbsPos.y)
      } else {
        // 重新计算 更新矩形 不过说真的 这里好像不怎么需要
        this.renderSelectionArea(this.initialRelPos, this.currentRelPos)
        // this.$emit('selecting')
      }
    },

```

### renderSelectionArea

实时更新 划过的矩形

```  js 
      renderSelectionArea (initPoint, endPoint) {
      const minX = Math.min(initPoint.x, endPoint.x)
      const maxX = Math.max(initPoint.x, endPoint.x)
      const minY = Math.min(initPoint.y, endPoint.y)
      const maxY = Math.max(initPoint.y, endPoint.y)

      this.$refs.selectionArea.style.left = minX + 'px'
      this.$refs.selectionArea.style.top = minY + 'px'
      this.$refs.selectionArea.style.width = maxX - minX + 'px'
      this.$refs.selectionArea.style.height = maxY - minY + 'px'
    },

```

### resizeElementBy

实时计算并更新缩放

``` js
    resizeElementBy (el, offX, offY) {
      const parentCompStyle = window.getComputedStyle(this.getParentMr(el))
      const elCompStyle = window.getComputedStyle(el)

      const parentH = parseInt(parentCompStyle.height)
      const parentW = parseInt(parentCompStyle.width)
      const elMinH = parseInt(elCompStyle.minHeight)
      const elMinW = parseInt(elCompStyle.minWidth)

      let newTop = parseInt(elCompStyle.top)
      let newLeft = parseInt(elCompStyle.left)
      let newRight = parseInt(elCompStyle.right)
      let newBottom = parseInt(elCompStyle.bottom)
      let newHeight = parseInt(elCompStyle.height)
      let newWidth = parseInt(elCompStyle.width)

      let diffX = offX
      let diffY = offY

      if (this.handle.indexOf('t') !== -1) {
        if (newHeight - offY < elMinH) diffY = (newHeight - elMinH)
        else if (newTop + offY < 0) diffY = (0 - newTop)
        newTop += diffY
        newHeight -= diffY
      }
      if (this.handle.indexOf('l') !== -1) {
        if (newWidth - offX < elMinW) diffX = (newWidth - elMinW)
        else if (newLeft + offX < 0) diffX = (0 - newLeft)
        newLeft += diffX
        newWidth -= diffX
      }
      if (this.handle.indexOf('b') !== -1) {
        if (newHeight + offY < elMinH) diffY = (elMinH - newHeight)
        else if (newTop + newHeight + offY > parentH) diffY = (parentH - newTop - newHeight)
        newHeight += diffY
        newBottom -= diffY
      }
      if (this.handle.indexOf('r') !== -1) {
        if (newWidth + offX < elMinW) diffX = (elMinW - newWidth)
        else if (newLeft + newWidth + offX > parentW) diffX = (parentW - newLeft - newWidth)
        newWidth += diffX
        newRight -= diffX
      }

      el.style.height = (el.style.height !== 'auto') ? newHeight + 'px' : 'auto'
      el.style.width = (el.style.width !== 'auto') ? newWidth + 'px' : 'auto'
      el.style.top = (el.style.top !== 'auto') ? newTop + 'px' : 'auto'
      el.style.left = (el.style.left !== 'auto') ? newLeft + 'px' : 'auto'
      el.style.bottom = (el.style.bottom !== 'auto') ? newBottom + 'px' : 'auto'
      el.style.right = (el.style.right !== 'auto') ? newRight + 'px' : 'auto'
    },

```

### moveElementBy

元素距离-实时更新

``` js
    moveElementBy (el, offX, offY) {
      const elCompStyle = window.getComputedStyle(el)

      // Re-set height and width on move to preserve dimensions (due addition of bottom/right props)
      el.style.height = el.style.height
      el.style.width = el.style.width

      el.style.top = (el.style.top !== 'auto')
        ? this.fixPosition(el, parseInt(elCompStyle.top) + offY, 'top') + 'px'
        : 'auto'
      el.style.left = (el.style.left !== 'auto')
        ? this.fixPosition(el, parseInt(elCompStyle.left) + offX, 'left') + 'px'
        : 'auto'
      el.style.bottom = (el.style.bottom !== 'auto')
        ? this.fixPosition(el, parseInt(elCompStyle.bottom) - offY, 'bottom') + 'px'
        : 'auto'
      el.style.right = (el.style.right !== 'auto')
        ? this.fixPosition(el, parseInt(elCompStyle.right) - offX, 'right') + 'px'
        : 'auto'
    },

```

### fixPosition

如果不是 `auto` 需要计算

``` js
    fixPosition (el, val, prop) {
      if (val < 0) return 0

      const parentCompStyle = window.getComputedStyle(this.getParentMr(el))
      const elCompStyle = window.getComputedStyle(el)

      if ((prop === 'top' || prop === 'bottom') && (val + parseInt(elCompStyle.height) > parseInt(parentCompStyle.height))) {
        return parseInt(parentCompStyle.height) - parseInt(elCompStyle.height)
      }
      if ((prop === 'left' || prop === 'right') && (val + parseInt(elCompStyle.width) > parseInt(parentCompStyle.width))) {
        return parseInt(parentCompStyle.width) - parseInt(elCompStyle.width)
      }
      return val
    },

```

###getParentMr

获得父辈,  画布 或者 另一个 mrEl 容器

``` js
    getParentMr (element) {
      let parentMr = null
      let currentMr = element

      while (parentMr === null) {
        if (currentMr === null || currentMr.parentElement === null) break
        else if (currentMr.dataset.mrContainer) parentMr = currentMr // 画布
        else if (currentMr.parentElement.dataset.mrEl) parentMr = currentMr.parentElement
        // 另一个 mrEl 容器
        currentMr = currentMr.parentElement
      }
      return parentMr
    },

    getMouseAbsPoint (e) {
      return {x: e.clientX, y: e.clientY}
    },

    getMouseRelPoint (e) {
      const mainContainer = document.getElementById('main')
      const x = e.clientX + mainContainer.scrollLeft - mainContainer.offsetLeft - this.$el.offsetLeft
      const y = e.clientY + mainContainer.scrollTop - mainContainer.offsetTop - this.$el.offsetTop

      return {x, y}
    },

```

### moveStopData

整理 元素 位置 和 鼠标位置


``` js
    moveStopData () {
      return {
        moveElData: this.mrElements.map(el => {
          return {
            elId: el.childNodes[0].id,
            top: (el.style.top.indexOf('%') !== -1 || el.style.top === 'auto')
              ? el.style.top
              : parseInt(el.style.top),
            left: (el.style.left.indexOf('%') !== -1 || el.style.left === 'auto')
              ? el.style.left
              : parseInt(el.style.left),
            bottom: (el.style.bottom.indexOf('%') !== -1 || el.style.bottom === 'auto')
              ? el.style.bottom
              : parseInt(el.style.bottom),
            right: (el.style.right.indexOf('%') !== -1 || el.style.right === 'auto')
              ? el.style.right
              : parseInt(el.style.right)
          }
        }),
        relMouseX: this.currentRelPos.x,
        relMouseY: this.currentRelPos.y,
        absMouseX: this.currentAbsPos.x,
        absMouseY: this.currentAbsPos.y
      }
    },

```

### resizeStopData

每个 选择元素 的ID和位置数据

``` js
    resizeStopData () {
      return this.mrElements.map(el => {
        return {
          elId: el.childNodes[0].id,
          top: (el.style.top.indexOf('%') !== -1 || el.style.top === 'auto')
            ? el.style.top
            : parseInt(el.style.top),
          left: (el.style.left.indexOf('%') !== -1 || el.style.left === 'auto')
            ? el.style.left
            : parseInt(el.style.left),
          bottom: (el.style.bottom.indexOf('%') !== -1 || el.style.bottom === 'auto')
            ? el.style.bottom
            : parseInt(el.style.bottom),
          right: (el.style.right.indexOf('%') !== -1 || el.style.right === 'auto')
            ? el.style.right
            : parseInt(el.style.right),
          height: (el.style.height.indexOf('%') !== -1 || el.style.height === 'auto')
            ? el.style.height
            : parseInt(el.style.height),
          width: (el.style.width.indexOf('%') !== -1 || el.style.width === 'auto')
            ? el.style.width
            : parseInt(el.style.width)
        }
      })
    },

```

### selectStopData

用实时更新的 矩形 规范 数据

``` js
    selectStopData () {
      return {
        top: parseInt(this.$refs.selectionArea.style.top),
        bottom: parseInt(this.$refs.selectionArea.style.height) + parseInt(this.$refs.selectionArea.style.top),
        left: parseInt(this.$refs.selectionArea.style.left),
        right: parseInt(this.$refs.selectionArea.style.width) + parseInt(this.$refs.selectionArea.style.left)
      }
    }
  }
}
```