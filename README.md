# 项目介绍
这是一个基于Express框架node.js的博客页面。分为前台注册、登录、文章列表、评论等页面。后台分为用户管理、栏目管理、文章管理等页面。

## 项目目录

```
 blog 项目
  │-- db        数据库储存目录
  │-- models    数据库模型文件目录
  │-- public    公共文件目录
  │-- routers   路由文件目录
  │-- schemas   数据结构文件目录
  │-- views     模板视图文件目录
  │-- app.js    应用启动入口文件
  │-- package.json
```


下面是本项目涉及到的一些知识点，代码内容和项目无关。

# 一、mongodb

安装

```shell
$ npm install mongoose
```

## mongodb 数据库的连接

```
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/my_database');
```

进入到你本地数据库的 `bin` 目录下；

```shell
...\mongodb\Server\3.5\bin
```

目录下设置你本地的项目路径和数据库的端口。

```shell
mongod --dbpath=...\db --port=27018
```

路径指向本地项目中的db目录。

## 定义模型

```
var Schema = mongoose.Schema,
    ObjectId = Schema.ObjectId;
 
var BlogPost = new Schema({
    author    : ObjectId,
    title     : String,
    body      : String,
    date      : Date
});
```

## mongoose 操作文档

http://mongoosejs.com/docs/guide.html

# 二、express 框架

安装 `express` 

```shell
$ npm install express --save
```

通过应用生成器工具 `express` 可以快速创建一个应用的骨架。

```shell
$ npm install express-generator -g
```

在当前工作目录下创建一个命名为 `blog` 的应用

```shell
$ express blog

   create : blog
   create : blog/package.json
   create : blog/app.js
   create : blog/public
   create : blog/public/javascripts
   create : blog/public/images
   create : blog/routes
   create : blog/routes/index.js
   create : blog/routes/users.js
   create : blog/public/stylesheets
   create : blog/public/stylesheets/style.css
   create : blog/views
   create : blog/views/index.jade
   create : blog/views/layout.jade
   create : blog/views/error.jade
   create : blog/bin
   create : blog/bin/www
```

然后安装所有依赖包：

```shell
$ cd myapp 
$ npm install
```

启动这个应用( windows 平台下 )

```shell
> set DEBUG=myapp & npm start
```

通过 Express 应用生成器创建的应用一般都有如下目录结构：

```
  .
  ├── app.js
  ├── bin
  │   └── www
  ├── package.json
  ├── public
  │   ├── images
  │   ├── javascripts
  │   └── stylesheets
  │       └── style.css
  ├── routes
  │   ├── index.js
  │   └── users.js
  └── views
      ├── error.jade
      ├── index.jade
      └── layout.jade
```

本项目的目录和该目录略有不同

创建一个 Express 应用

```javascript
var express = require('express');
var app = express();
```
项目中的app对象，就是express的一个实例


## 静态文件

通过 Express 内置的 `express.static` 托管静态文件。

当用户访问的`url`以`/public`开始，那么直接返回对应`__dirname + '/public'`下的文件

```
app.use( '/public', express.static( __dirname + '/public') );
```


## 路由

路由（Routing）是由一个 URI（或者叫路径）和一个特定的 HTTP 方法（GET、POST 等）组成的。每一个路由都可以有一个或者多个处理器函数，当匹配到路由时，这个/些函数将被执行。

路由的定义由如下结构组成：

> `app.METHOD(PATH, HANDLER)`

其中，`app` 是 `express` 实例；`METHOD` 是某个 `HTTP` 的一个请求方式；`PATH`是服务器端的路径；`HANDLER` 是当路由匹配到时需要执行的函数。

```
app.get('/', function(req, res) {
  res.send('hello world');
});
```

使用多个回调函数处理路由

```
app.get('/example/b', function (req, res, next) {
  console.log('响应将由下一个函数发送...');
  next();
}, function (req, res) {
  res.send('Hello from B!');
});
```

### 响应方法

- `res.render(view [, locals] [, callback])` 

`locals` 为模板传入数据；

- res.send([body])

```
res.send(new Buffer('whoop'));
res.send({ some: 'json' });
res.send('<p>some html</p>');
res.status(404).send('Sorry, we cannot find that!');
res.status(500).send({ error: 'something blew up' });
```

### express.Router

`express.Router` 类创建模块化、可挂载的路由句柄。Router 实例是一个完整的中间件和路由系统。

## 中间件

简单说，中间件（middleware）就是处理HTTP请求的函数。它最大的特点就是，一个中间件处理完，再传递给下一个中间件。

### 应用级中间件

应用级中间件绑定到 app 对象 使用 app.use() 和 app.METHOD()， 其中， METHOD 是需要处理的 HTTP 请求的方法，例如 GET, PUT, POST 等等，全部小写。


### 路由级中间件

上面的 `express.Router` 就是一个路由中间件

```
var app = express();
var router = express.Router();

// 没有挂载路径的中间件，通过该路由的每个请求都会执行该中间件
router.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next();
});

// 一个中间件栈，显示任何指向 /user/:id 的 HTTP 请求的信息
router.use('/user/:id', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();
}, function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 一个中间件栈，处理指向 /user/:id 的 GET 请求
router.get('/user/:id', function (req, res, next) {
  // 如果 user id 为 0, 跳到下一个路由
  if (req.params.id == 0) next('route');
  // 负责将控制权交给栈中下一个中间件
  else next(); //
}, function (req, res, next) {
  // 渲染常规页面
  res.render('regular');
});

// 处理 /user/:id， 渲染一个特殊页面
router.get('/user/:id', function (req, res, next) {
  console.log(req.params.id);
  res.render('special');
});

// 将路由挂载至应用
app.use('/', router);
```

### 错误处理中间件

```
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

### use方法

`use`是`express`注册中间件的方法，它返回一个函数。

```
var express = require("express");
var http = require("http");

var app = express();

app.use(function(request, response, next) {
  console.log("In comes a " + request.method + " to " + request.url);
  next();
});

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

http.createServer(app).listen(1337);
```

上面代码使用`app.use`方法，注册了两个中间件。收到`HTTP`请求后，先调用第一个中间件，在控制台输出一行信息，然后通过`next`方法，将执行权传给第二个中间件，输出`HTTP`回应。由于第二个中间件没有调用`next`方法，所以`request`对象就不再向后传递了。

参考：

http://www.expressjs.com.cn/guide/using-middleware.html

http://javascript.ruanyifeng.com/nodejs/express.html#

