### 初始化后台
```
npm init -y
npm i express mongoose --save
npm i nodemon -g
```
### 创建应用结构
```
|- models
|- public
|- routers
|- views
|- server.js
|- README.md
```
### user集合数据 models/UserModel.js
```
// 1.引入mongoose
const mongoose = require('mongoose')

// 2.字义Schema(描述文档结构)
const userSchema = new mongoose.Schema({
  username: {type: String, required: true}, // 用户名
  password: {type: String, required: true}, // 密码
})

// 3. 定义Model(与集合对应, 可以操作集合)
const UserModel = mongoose.model('users', userSchema)

// 4. 向外暴露Model
module.exports = UserModel
```
### 后台页面 public/index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>后台首页</title>
</head>
<body>
  <h2>欢迎访问应用后台</h2>
</body>
</html>
```
### 后端路由路由器页面 routers/index.js
```
const express = require('express')
const router = express.Router()

// 定义路由
router.get('/hello', (req, res) => {
  res.send('Hello express router!')
})

module.exports = router
```
### 启动模块 server.js
```
/*
应用的启动模块
1. 通过express启动服务器
2. 通过mongoose连接数据库
  说明: 只有当连接上数据库后才去启动服务器
3. 使用中间件
 */
const mongoose = require('mongoose')
const express = require('express')

const app = express()

// 声明使用静态中间件
app.use(express.static('public'))
// 声明使用解析post请求的中间件
app.use(express.urlencoded({extended: true})) // 请求体参数是: name=tom&pwd=123
// 声明使用路由器中间件
const indexRouter = require('./routers')
app.use('/', indexRouter)

// 通过mongoose连接数据库
mongoose.connect('mongodb://localhost/gzhipin5', {useNewUrlParser: true})
  .then(() => {
    console.log('连接数据库成功!!!')
    // 只有当连接上数据库后才去启动服务器
    app.listen('5000', () => {
      console.log('服务器启动成功, 请访问: http://localhost:5000')
    })
  })
  .catch(error => {
    console.error('连接数据库失败', error)
  })
```
### 配置启动命令/运行/访问 package.json
```
"scripts": {
  "start": "nodemon server.js"
},
```
https://www.mongodb.org/dl/win32/x86_64
mongodb-win32-x86_64-3.2.4-signed
