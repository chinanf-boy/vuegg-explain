# Vuegg

「 🐣 vue GUI generator [vuegg.github.io](vuegg.github.io) 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    
Explanation

> "version": "0.17.0"

[github source](https://github.com/vuegg/vuegg)

~~[english](./README.en.md)~~

---

本次解释的项目是一个`全栈`项目, 所以分 server/client

- [ ] [server](#server)


- [ ] [client](#client)


---

本目录

---

### server

> 先说说, server 做了什么

1. `get-access-token`

从`github` - `oauth app` 中获取, 用户信息-`access_token`

2. `save-vuegg-project`

将, 前端所有状态保存在`github用户`-自己的`repo`中, 就像新建一个`repo` { `vue.gg` 文件被保存}

3. `get-vuegg-project`

获取上面的项目, {`vue.gg`文件}

4. `generate`

将所有前端-状态转译生成`.zip`文件, 下载

[>>> server <=== 先这个](./server.md)

### client

> 那么, 前端又做了什么呢

[>>> client](./client.md#client)

那就多了,

主要两大块

#### 1. `/` - 编辑页面

> 要可视化组件, 自然而然需要 

- 拖拽操作的 `元素栏` => 画笔

- 每个元素设置的 `设置栏`

- 添加页面的 `页面栏`

> 有了画笔, 自然需要画布 

- 放下操作的 `画布栏` => 画布

> 专注画布, 也提供了快捷键 比如 `Ctrl + Z` 自然就是回退等操作了

- 基本操作 「回退/前进/删除/保存/上传等」 `操作导航栏`

- 自然少不了 `用户栏`


#### 2. `/preview` - 预览页面

> 自然就是路由转换, 组件变换 的 预览

- 三种边窗大小变化对应 一般 {`移动端|平板|桌面`}

- 少不了, `回退 等` 小按钮的操作

#### 3. 404 页面

可以做得萌一点噢
