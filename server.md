## server

`vuegg/server`

---

---

### package.json

``` js
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "NODE_ENV=development nodemon server.js"
```

### server.js

`vuegg/server/server.js`

``` js
const Koa = require('koa')
const body = require('koa-body') // koa body parser middleware
const logger = require('koa-logger') // A Koa middleware for reporting JSON access logs
const historyFallback = require('koa2-history-api-fallback') // Fallback to index.html for applications that are using the HTML 5 history API
const router = require('koa-router')({ prefix: '/api' })
// 全部路由前缀加 `/api`
const static = require('koa-static')(path.resolve(__dirname, '..', 'client','dist'))
// 将client 的生产构建拉过来

```

可以看出, 是 `Koa` 框架起作用啦

`Koa`的中间件模式, 让一个一个的请求和响应过滤


``` js
// Routes definition
router.post('/get-access-token', getAccessToken)
router.post('/save-vuegg-project', saveVueggProject)
router.get('/get-vuegg-project', getVueggProject)
router.post('/generate', generate)
// 这里就说明了, 做了什么

// Middleware
app.use(body())
app.use(logger())
app.use(historyFallback())
app.use(static)
app.use(router.routes())
app.use(router.allowedMethods())
//返回单独的中间件，用于响应带OPTIONS请求的Allow头，该头包含允许的方法，并根据需要响应405 Method Not Allowed和501 Not Implemented。
```

---

- [getAccessToken](#getAccessToken)
- [saveVueggProject](#saveVueggProject)
- [getVueggProject](#getVueggProject)

> 上面三种其实是运用[octokit use](https://github.com/octokit/rest.js) `GitHub REST API client for Node.js`

- [generate](#generate) 最为复杂的

> 涉及到, 前端状态本身的数据, 转译阿, 变成`Vue`啊等等

> 所以这段, 应该对`client`有个概括性认识后再来吧

[直通车-client](./readme.md#client)

---

### 1. octokit-use

<details>

#### getAccessToken

``` js
async function getAccessToken (ctx) {
  try {
    let resp = await auth.getAccessToken(ctx.request.body)
    ctx.response.status = 200
    ctx.response.type = 'text/plain'
    ctx.response.body = resp
  } catch (e) {
    console.error('\n> Could not obtain token\n' + e)
    process.exit(1)
  }
}
```

`server/auth/index.js`

``` js
async function getAccessToken ({code, state}) {
  const options = {
    method: 'POST',
    uri: 'https://github.com/login/oauth/access_token',
    form: {
      client_id: process.env.CLIENT_ID,
      client_secret: process.env.CLIENT_SECRET,
      // <==== 这里是应用的密钥, 作者并没有保存, 所以开发模式会请求不了, 用户信息access-token的
      code: code // 用户信息请求
    }
  }

  try {
    // rp == request-promise-native 服务器http请求
    let resp = await rp(options)
    // qs == querystring 解析uri
    return qs.parse(resp).access_token // 用户token
  } catch (e) {
    console.error("get not access_token")
  }
}
```

#### saveVueggProject

保存项目分两段

1. 用户的项目的存在是否

2. 对存在项目文件的操作

<details>

``` js
async function saveVueggProject (ctx) {
  try {
    let resp = await github.saveVueggProject(ctx.request.body)
    if (resp) {
      ctx.response.status = 200
      ctx.response.body = resp
      ctx.response.type = 'text/plain'
    }
  } catch (e) {
    console.error('\n> Could not save the project definition\n' + e)
    process.exit(1)
  }
}
```

`server/github/index.js`

1. 用户的项目的存在是否

``` js
// 保存项目
async function saveVueggProject ({project, owner, repo, token}) {
// 1. 用户的项目的存在是否
  let existingRepo = await getRepo(owner, repo, token)
  if (!existingRepo) { await createRepo(repo, token) }


// 2. 对存在项目文件的操作
  return await saveFile(project, owner, repo, 'vue.gg', token)
}

// 有没有存在的项目呢
async function getRepo (owner, repo, token) {
  // octokit.authenticate({type: 'oauth', token})

  try {
    return await octokit.repos.get({owner, repo})
  } catch (e) {
    console.log('(REPO) - ' + owner + '/' + repo + '  does not exist')
    return false
  }
}

// 创建新的
async function createRepo (name, token) {
  octokit.authenticate({type: 'oauth', token})

  try {
    return await octokit.repos.create({name, license_template: 'mit'})
  } catch (e) {
    console.error('(REPO) - Failed to create: ' + owner + '/' + name)
    console.error(e)
    return false
  }
}

```

2. 对存在项目文件的操作

``` js
// 获得项目文件`vue.gg`内容
async function getContent (owner, repo, path, token) {
  // octokit.authenticate({type: 'oauth', token})

  try {
    return await octokit.repos.getContent({owner, repo, path})
  } catch (e) {
    console.error('(FILE) - ' + owner + '/' + repo + '/' + path + ' does not exist')
    return false
  }
}

// 2. 对存在项目文件的操作
async function saveFile (content, owner, repo, path, token) {
  let existentFile = await getContent(owner, repo, path, token)

  let ghData = {
    owner, repo, path,
    content: Buffer.from(JSON.stringify(content)).toString('base64'),
    committer: {name: 'vuegger', email: '35027416+vuegger@users.noreply.github.com'}
  }

  octokit.authenticate({type: 'oauth', token})

  try {
    if (!existentFile) { // 项目中没有相应文件
    // 创建
      console.log('(FILE) - creating: ' + owner + '/' + repo + '/' + path)
      return await octokit.repos.createFile({...ghData, message: 'vuegg save'})
    } else {
    // 更新  
      console.log('(FILE) - updating: ' + owner + '/' + repo + '/' + path)
      return await octokit.repos.updateFile({...ghData, message: 'vuegg update', sha: existentFile.data.sha})
    }
  } catch (e) {
    console.error('(FILE) - Failed to save: ' + owner + '/' + repo + '/' + path)
    return false
  }
}


```
</details>

#### getVueggProject

``` js
// 获取项目中文件`vue.gg`内容
async function getVueggProject (ctx) {
  let params = ctx.request.query
  try {
    let resp = await github.getContent(params.owner, params.repo, 'vue.gg', params.token)
    if (resp) {
      ctx.response.status = 200
      ctx.response.body = resp
      ctx.response.type = 'application/json'
    }
  } catch (e) {
    console.error('\n> Could not fetch the project definition\n' + e)
    process.exit(1)
  }
}
```

</details>

#### 2. generate

``` js
async function generate (ctx) {
  try {
    let zipFile = await generator(ctx.request.body, ROOT_DIR)
    if (zipFile) {
      console.log('> Download -> ' + zipFile)
      ctx.response.status = 200
      ctx.response.type = 'zip'
      ctx.response.body = fs.createReadStream(zipFile)
    }
  } catch (e) {
    console.error('\n> Could not complete the project generation...\n' + e)
    process.exit(1)
  }
}

```
