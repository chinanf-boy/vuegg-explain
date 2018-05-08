## mrEl


### handles

缩放 class 名称不同

``` js
    <div data-mr-handle="true"
      v-if="resizable"
      v-for="handle in handles"
      :key="handle"
      class="handle" :class="handle"
      :style="{ display: active ? 'block' : 'none'}">
    </div>
```

``` js
    handles: {
      type: Array,
      default: function () {
        return ['mt', 'mr', 'mb', 'ml', 'tl', 'tr', 'br', 'bl']
      }
    }
```