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

[>>> server](./server.md)

### client

