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

### 