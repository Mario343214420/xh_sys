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
### 后台接口测试
#### 1)需求:
a.后台应用运行端口指定为5000<br>
b.提供一个用户注册的接口<br>
a)path为: /register<br>
b)请求方式为: POST<br>
c)接收username和password参数<br>
d)admin是已注册用户<br>
e)注册成功返回: {code: 0, data: {_id: 'abc', username: ‘xxx’, password:’123’}}<br>
f)注册失败返回: {code: 1, msg: '此用户已存在'}<br>
#### 2)注册新路由: routes/index.js
```
router.post('/register', function (req, res, next) {
  const {username, password} = req.body
  console.log('register', username, password)
  if (username === 'admin') {
    res.send({code: 1, msg: '此用户已存在'})
  } else {
    res.send({code: 0, data: {_id: 'abc', username, password}})
  }
})
```
#### 3)postman测试接口
http://localhost:5000/register<br>
Body:<br>
radio:x-www-form-urlencoded
type:post

KEY  | VALUE
 ---- | -----
 username  | admin
 password  | 123456

RES:返回对应请求内容

### 注册/登陆后台处理
#### 读取保存 请求数据 加密依赖库
```
npm i cookie-parser blueimp-md5 --save
```
#### 【Mongo Plugin】 Webstrom插件
Plugins安装【Mongo Plugin】<br>
右侧配置对应数据库接口<br>
mongod启动
#### 操作users的Model: models/UserModel.js
```
/*
能操作users集合数据的Model
 */
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
#### 路由器模块: routes/index.js
```
/*
用来定义路由的路由器模块
 */
const express = require('express')
const md5 = require('blueimp-md5')
const UserModel = require('../models/UserModel')

// 得到路由器对象
const router = express.Router()

// 指定需要过滤的属性
const filter = {password: 0, __v: 0}

// 注册的路由
router.post('/register', (req, res) => {
  // 读取请求参数数据
  const {username, password, type} = req.body
  // 处理: 判断用户是否已经存在, 如果存在, 返回提示错误的信息, 如果不存在, 保存
  // 查询(根据username)
  UserModel.findOne({username})
    .then(user => {
      // 如果user有值(已存在)
      if (user) {
        // 返回提示错误的信息
        res.send({code: 1, msg: '此用户已存在'})
      } else { // 没值(不存在)
        // 保存
        return UserModel.create({username, type, password: md5(password)})
      }
    })
    .then(user => {
      console.log('create()', user)
      // 生成一个cookie(userid: user._id), 并交给浏览器保存
      res.cookie('userid', user._id, {maxAge: 1000 * 60 * 60 * 24})
      // 返回包含user的json数据
      const data = {username, type, _id: user._id} // 响应数据中不要携带password
      res.send({code: 0, data})
    })
    .catch(error => {
      console.error('注册异常', error)
      res.send({code: 1, msg: '注册异常, 请重新尝试'})
    })
})

// 登陆的路由
router.post('/login', (req, res) => {
  const {username, password} = req.body
  // 根据username和password查询数据库users, 如果没有, 返回提示错误的信息, 如果有, 返回登陆成功信息(包含user)
  UserModel.findOne({username, password: md5(password)}, filter)
    .then(user => {
      if (user) { // 登陆成功
        // 生成一个cookie(userid: user._id), 并交给浏览器保存
        res.cookie('userid', user._id, {maxAge: 1000 * 60 * 60 * 24})
        // 返回登陆成功信息(包含user)
        res.send({code: 0, data: user})
      } else {// 登陆失败
        res.send({code: 1, msg: '用户名或密码不正确!'})
      }
    })
    .catch(error => {
      console.error('登陆异常', error)
      res.send({code: 1, msg: '登陆异常, 请重新尝试'})
    })
})

module.exports = router
```
#### server.js
```
app.use(express.json()) // 请求体参数是json结构: {name: tom, pwd: 123}
// 声明使用解析cookie数据的中间件
const cookieParser = require('cookie-parser')
app.use(cookieParser())
```
#### postman测试登录注册
