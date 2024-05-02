# Node.js 登录功能实现

在这篇笔记中，我将详细解释一个使用 Node.js 和 Express 构建的简单用户登录功能的实现。

## 1. 导入依赖模块

```javascript
const express = require("express");
const hash = require("pbkdf2-password")();
const path = require("path");
const session = require("express-session");
```

## 2. 创建 Express 应用并进行配置

```javascript
const app = (module.exports = express());

app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));
```

## 3. 添加中间件

```javascript
app.use(express.urlencoded({ extended: false }));
app.use(
  session({
    resave: false,
    saveUninitialized: false,
    secret: "shhhh, very secret",
  })
);
```

## 4. 添加 session 消息处理中间件

```javascript
app.use(function (req, res, next) {
  // ...
  next();
});
```

## 5. 创建虚拟数据库并存储用户信息

```javascript
const users = {
  tj: { name: "tj" },
};

hash({ password: "foobar" }, function (err, pass, salt, hash) {
  users.tj.salt = salt;
  users.tj.hash = hash;
});
```

## 6. 创建认证函数

```javascript
function authenticate(name, pass, fn) {
  // ...
}
```

## 7. 创建限制访问的中间件

```javascript
function restrict(req, res, next) {
  // ...
}
```

## 8. 定义路由和请求处理器

```javascript
app.get("/", function (req, res) {
  res.redirect("/login");
});

app.get("/restricted", restrict, function (req, res) {
  res.send('Wahoo! restricted area, click to <a href="/logout">logout</a>');
});

app.get("/logout", function (req, res) {
  req.session.destroy(function () {
    res.redirect("/");
  });
});

app.get("/login", function (req, res) {
  res.render("login");
});

app.post("/login", function (req, res, next) {
  // ...
});
```

## 9. 启动服务器

```javascript
if (!module.parent) {
  app.listen(3000);
  console.log("Express started on port 3000");
}
```

以上就是一个基础的用户登录系统的实现。实际的生产环境中会有更多的复杂性和安全性考虑，例如使用 HTTPS，防止 SQL 注入，防止跨站脚本攻击（XSS），防止跨站请求伪造（CSRF），等等。

## 附录

### 1. Express.js 中的 `app.get()` 和 `restrict` 函数

#### `app.get()`

在 Express 中，`app.get()` 方法用于定义一个处理 GET 请求的路由。它的第一个参数是一个路径，第二个参数是一个或多个中间件或请求处理器。

例如，`app.get('/restricted', restrict, function(req, res){...})` 定义了一个处理 GET `/restricted` 请求的路由。这个路由有两个中间件：`restrict` 和一个匿名函数。

#### `restrict` 函数

`restrict` 是一个自定义的中间件，它的作用是检查用户是否已经登录。如果用户已经登录（即 `req.session.user` 为真），那么调用 `next()` 函数，表示这个中间件已经完成了它的工作，Express 可以调用下一个中间件或请求处理器了。在这个例子中，下一个中间件就是那个匿名函数。所以，如果用户已经登录，那么 Express 就会调用那个匿名函数，返回一个包含 "Wahoo! restricted area, click to <a href="/logout">logout</a>" 的响应。

如果用户没有登录（即 `req.session.user` 为假），那么 `restrict` 中间件会在 session 中设置一个 error 消息，然后重定向到登录页面。注意，它没有调用 `next()` 函数，所以 Express 不会调用下一个中间件或请求处理器。在这个例子中，那个匿名函数就不会被调用。

#### `next()` 函数

`next()` 函数的作用是告诉 Express 这个中间件已经完成了它的工作，可以调用下一个中间件或请求处理器了。如果一个中间件没有调用 `next()` 函数，那么后续的中间件和请求处理器就不会被调用。

### 2. Express-Session 中间件工作原理

`express-session` 是一个用于 Express.js 的会话管理中间件。它允许我们在多个 HTTP 请求之间存储和访问数据。

#### 会话创建和更新

无论是 GET、POST 还是其他任何类型的 HTTP 请求，只要你的代码修改了会话数据（例如，通过 `req.session.xx = value`），那么 `express-session` 就会创建或更新会话。

`express-session` 中间件的工作方式是，当请求到达时，它会查看请求的 cookie 中是否有会话 ID。如果有，它会尝试从存储中获取与该 ID 对应的会话数据。如果没有，或者存储中没有与该 ID 对应的会话数据，它就会创建一个新的会话。

然后，当响应被发送时，如果会话数据被修改了，`express-session` 就会更新存储中的会话数据，并在响应的 cookie 中设置新的会话 ID。如果会话数据没有被修改，`express-session` 就不会更新存储中的会话数据，也不会在响应的 cookie 中设置新的会话 ID。

因此，无论请求的类型是什么，只要你的代码在处理请求时修改了会话数据，`express-session` 都会创建或更新会话。
