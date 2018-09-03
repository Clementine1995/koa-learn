# Koa

## 一、基本用法

### 1.1 架设 HTTP 服务

主要方法listen()

```javascript
const Koa = require('koa');
const app = new Koa();
app.listen(3000);
```

### 1.2 Context 对象

Koa 提供一个 Context 对象，表示一次对话的上下文（包括 HTTP 请求和 HTTP 回复）。通过加工这个对象，就可以控制返回给用户的内容。
Context.response.body属性就是发送给用户的内容。请看下面的例子

```javascript
const Koa = require('koa');
const app = new Koa();

const main = ctx => {
  ctx.response.body = 'Hello World';
};

app.use(main);
app.listen(3000);
```

上面代码中，main函数用来设置ctx.response.body。然后，使用app.use方法加载main函数。
你可能已经猜到了，ctx.response代表 HTTP Response。同样地，ctx.request代表 HTTP Request。

### 1.3 HTTP Response 的类型

Koa 默认的返回类型是text/plain，如果想返回其他类型的内容，可以先用ctx.request.accepts判断一下，
客户端希望接受什么数据（根据 HTTP Request 的Accept字段），然后使用ctx.response.type指定返回类型。

```javascript
const main = ctx => {
  if (ctx.request.accepts('xml')) {
    ctx.response.type = 'xml';
    ctx.response.body = '<data>Hello World</data>';
  } else if (ctx.request.accepts('json')) {
    ctx.response.type = 'json';
    ctx.response.body = { data: 'Hello World' };
  } else if (ctx.request.accepts('html')) {
    ctx.response.type = 'html';
    ctx.response.body = '<p>Hello World</p>';
  } else {
    ctx.response.type = 'text';
    ctx.response.body = 'Hello World';
  }
};
```

### 1.4 网页模板

实际开发中，返回给用户的网页往往都写成模板文件。我们可以让 Koa 先读取模板文件，然后将这个模板返回给用户。

```javascript
const fs = require('fs');
const main = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = fs.createReadStream('./demos/template.html');
};
```

## 二、路由

### 2.1 原生路由

网站一般都有多个页面。通过ctx.request.path可以获取用户请求的路径，由此实现简单的路由。

```javascript
const main = ctx => {
  if (ctx.request.path !== '/') {
    ctx.response.type = 'html';
    ctx.response.body = '<a href="/">Index Page</a>';
  } else {
    ctx.response.body = 'Hello World';
  }
};
```

### 2.2 koa-route 模块

原生路由用起来不太方便，我们可以使用封装好的koa-route模块。

```javascript
const route = require('koa-route');

const about = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = '<a href="/">Index Page</a>';
};

const main = ctx => {
  ctx.response.body = 'Hello World';
};

app.use(route.get('/', main));
app.use(route.get('/about', about));
```

上面代码中，根路径/的处理函数是main，/about路径的处理函数是about。

### 2.3 静态资源

如果网站提供静态资源（图片、字体、样式表、脚本......），为它们一个个写路由就很麻烦，也没必要。koa-static模块封装了这部分的请求。

```javascript
const path = require('path');
const serve = require('koa-static');

const main = serve(path.join(__dirname));
app.use(main);
```

### 2.4 重定向

有些场合，服务器需要重定向（redirect）访问请求。比如，用户登陆以后，将他重定向到登陆前的页面。
ctx.response.redirect()方法可以发出一个302跳转，将用户导向另一个路由。