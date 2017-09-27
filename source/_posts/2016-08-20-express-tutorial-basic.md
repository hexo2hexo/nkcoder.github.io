title: 学习Express框架一：基础
categories: Backend
date: 2016-08-20 06:59:43
tags: [nodejs, express]
---


[express](http://expressjs.com/)是nodejs的一个流行的web框架。本文主要介绍将express作为服务端对外提供API接口时，需要了解的入门知识。Express版本：`4.x`。

# 1. Hello World

首先安装node（如果已安装，则略过）：

    ```
    $ brew install node
    ```

创建一个项目，然后安装express：

    ```
    $ mkdir express-hello-world && cd express-hello-world
    $ npm init
    $ npm install express --save
    ```

`npm init`会提示输入一些配置信息，回车使用默认值即可，执行完后，当前目录下会自动创建`package.json`文件。

`npm install express --save`表示为当前项目安装express依赖，该依赖信息会保存在`package.json`文件中。

<!-- more -->

新建文件`index.js`文件， 输入以下内容：

    ```js
    var express =  require('express');
    var app = express();

    app.get('/', function(req, res) {
      res.send('hello, world!');
    });

    app.listen(3000);
    ```

运行：

    ```
    $ node index.js
    ```

浏览器访问：`http://localhost:3000/`，会输出"hello, world!"。

# 2. 中间件middleware

在express中，中间件（middleware）函数是一种特殊的函数，它可以访问一个http请求周期中的request对象、response对象，以及表示调用栈中的下一个中间件函数的引用，如：

    ```js
    function (req, res, next) {
        next();
    }
    ```

其中，`req`, `res`和`next`三个参数名是约定的，不要使用其它的变量名。中间件函数可以修改request和response，或者提前结束response，也可以调用`next()`表示将执行传递给调用栈中的下一个中间件函数。

如果当前中间件函数没有结束HTTP请求，则必须调用`next()`将执行传递给下一个中间件函数，否则请求会挂起。

使用`app.use()`加载中间件函数，中间件函数加载的顺序决定了它的执行顺序，即先加载，先执行。

在上面hello-world的例子中，我们增加两个简单的中间件函数，分别打印两条日志信息。在`var app = express();`的后面增加以下代码：

    ```js
    app.use(function (req, res, next) {
      console.log('in middleware one...');
      next();
    });

    app.use(function (req, res, next) {
      console.log('in middleware two...');
      next();
    });
    ```

执行后，终端会依次输出两个中间件函数中的日志信息。

express中的中间件可以分为以下几类：

- app级中间件
- router级中间件
- 错误处理中间件
- 内置中间件
- 第三方中间件

这里仅简要介绍一下主要的app级中间件和router级中间件。

app级中间件，即将中间件函数绑定到`app`对象（即使用`express()`得到的对象），通过`app.use()`或者`app.METHOD()`方法来加载中间件函数，其中`METHOD()`表示HTTP请求中的`GET/POST/PUT`等方法。上面的hello-world示例中就是app级中间件：

    ```js
    app.get('/', function(req, res) {
      res.send('hello, world!');
    });
    ```

router级中间件与app级中间件的用法基本一致，不同的是，它将中间件函数绑定到`express.Router()`对象，通过`router.use()`或者`router.METHOD()`方法来加载。比如上面的app级中间件，用router级中间件的方法改写如下：

    ```js
    var router = express.Router();
    router.get('/', function(req, res) {
      res.send('hello, world!');
    });

    app.use('/', router);
    ```

引入router级中间件的好处之一是解耦，通过router去分层，然后app加载router。

# 3. HTTP的req对象

## 3.1 req.params取路径参数

express中，路径参数使用命名参数的方式，比如路径是`/user/:id`，则使用`req.params.id`取参数`:id`的值，如：

    ```
    /user/:id
    GET /user/15
    req.params.id  => 15
    ```

## 3.2 req.query取查询参数

取查询参数，只需要通过`req.query`根据key取值即可，如：

    ```
    GET /search?name=Ketty&gender=male
    req.query.name => Ketty
    req.query.gender => male

    GET /search?user[name]=Ketty&user[age]=30
    req.query.user.name => Ketty
    req.query.user.age  => 30
    ```

## 3.3 req.body 

要取HTTP请求中的body内容，使用`req.body`，但是需要借助第三方module，如`body-parser`和`multer`，以下示例来自[express文档](http://expressjs.com/en/4x/api.html#req.body)：

    ```js
    var app = require('express')();
    var bodyParser = require('body-parser');
    var multer = require('multer'); // v1.0.5
    var upload = multer(); // for parsing multipart/form-data

    app.use(bodyParser.json()); // for parsing application/json
    app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded

    app.post('/profile', upload.array(), function (req, res, next) {
      console.log(req.body);
      res.json(req.body);
    });
    ```

## 3.4 req.get()

提取HTTP header中的信息，其中，key是不区分大小写的，如：

    ```js
    req.get('Content-Type');    // text/html
    req.get('content-type');    // text/html
    ```

# 4. HTTP的res对象

## 4.1 res.status()

该方法仅仅是设置状态码，返回response还需调用`send()/end()`等方法，如：

    ```js
    res.status(200).end();
    res.status(401).send("error: unauthorized!");
    ```

## 4.2 res.json()

返回json格式的信息，res会自动设置response的`Content-Type`为`application/json`，如：

    ```js
    res.status(401).json({"error": "unauthorized"});
    ```

## 4.3 res.send()

发送HTTP响应信息，参数可以是字符串、数组、Buffer对象等，会根据参数的类型自动设置header的`Content-Type`，如：

    ```js
    res.send(new Buffer("buffer info"));        // Content-Type: application/octet-stream
    res.send("<small>small text</small>");      // Content-Type: text/html
    res.send({message: "Welcome"});             // Content-Type: application/json
    ```

## 4.4. res.set()

设置HTTP的header信息，如：

    ```js
    res.set('Content-Type', 'application/pdf');
    res.setHeader('Content-Disposition', 'attachment; filename=cnb.pdf');
    ```

## 4.5 res.render()

使用模板引擎渲染页面，然后作为response返回。如果参数表示的文件名不带后缀，则会根据模板引擎的设置，自动推断后缀；如果文件名带后缀，则会加载该后缀对应的模板引擎模块。如

    ```js
    res.render('index');
    ```

如果默认的模板引擎是jade，则express会从对应的路径下查找index.jade文件并渲染。

# 参考

- [4.x API](http://expressjs.com/en/4x/api.html)
