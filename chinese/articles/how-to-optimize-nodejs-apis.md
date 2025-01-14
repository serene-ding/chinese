> -  原文地址：[How to Optimize Your Node.js API](https://www.freecodecamp.org/news/how-to-optimize-nodejs-apis/)
> -  原文作者：[Kayode Adeniyi](https://www.freecodecamp.org/news/author/mkbadeniyi/)
> -  译者：Papaya HUANG
> -  校对者：

![How to Optimize Your Node.js API](https://www.freecodecamp.org/news/content/images/size/w2000/2022/08/pexels-ann-marie-kennon-1296000.jpg)

在这篇文章中，我将讲解如何优化使用Node.js编写的API。

### 前提条件

想要充分了解本文内容，你必须了解以下概念：

-   Node.js的设置与安装
-   如何使用Node创建API
-   如何使用Postman
-   JavaScript的async/await工作原理
-   Redis的基础操作

## API优化到底指的是什么

优化包含了改善API的响应时间。响应时间越短，API的速度越快。

我将在本文分享一些技巧，帮助你缩短响应时间、降低延迟、管理错误和吞吐量，并且最大限度地减少CPU和内存的使用。

# 如何优化Node.js的API

## 1\. 始终使用异步函数

异步函数就像JavaScript的心脏。因此，优化CPU使用率的最佳方法就是编写异步函数来执行非阻塞I/O操作。

I/O操作包括对数据的读和写。它可以在数据库、云存储或者任何本地磁盘上进行。

在大量使用I/O操作的应用使用异步函数可以提高效率。因为由于没有阻塞I/O，当一个请求在做输入/输出操作的时候，CPU可以同时处理多个请求。
  
举例如下：

```js
var fs = require('fs');
// 执行阻塞I/O
var file = fs.readFileSync('/etc/passwd');
console.log(file);
// 执行非阻塞I/O
fs.readFile('/etc/passwd', function(err, file) {
    if (err) return err;
    console.log(file);
});
```

-   使用Node包**fs**来处理文件 
-   **readFileSync()**是同步函数，会在执行完成前阻塞线程
-   **readFile()**是异步函数，会立刻返回并在后台运行

## 2\. 避免在API中使用session和cookie，仅在API响应中发送数据

当我们使用cookie或者session来存储临时状态的时候，会占用非常多的服务器内存。

现在通用无状态API，并且也有JWT、OAuth等验证机制。验证令牌保存在客户端以便服务器管理状态。

JWT是基于JSON的用于API验证的安全令牌。JWT可以被看到，但一旦发送就无法修改。JWT只是一个序列并没有加密。 OAuth不是API或服务 - 相反，它是授权的开放标准。OAuth是一组用于获取令牌的标准步骤。

同时，也不要把时间浪费在使用Node.js来服务静态文件。这方面NGINX和Apache做得更好。

使用Node搭建API的时候，不要在响应中发送完整的HTML页面。当仅有数据通过API发送的时候，Node服务得会更好。大部分Node应用都使用JSON数据。

## 3\. 优化数据库查询

优化Node API的重要一环是优化查询。特别是对于大型应用来说，我们需要多次查询数据库，所以一个糟糕的查询会降低应用的整体性能。

索引是一种优化数据库性能的方法，通过最小化处理查询时所需的磁盘访问次数来实现。它是一种数据结构技术，用于快速定位和访问数据库中的数据。索引是使用几个数据库列创建的。

假设我们有一个没有索引的数据库模式，并且数据库包含100万条记录。与带有索引的模式相比，使用没有索引的模式做一个简单的find（查找）查询将扫描更多的记录来找到匹配的记录。

-   没有索引的查询

```js
> db.user.find({email: 'ofan@skyshi.com'}).explain("executionStats")
```

-  有索引的查询

```js
> db.getCollection("user").createIndex({ "email": 1 }, { "name": "email_1", "unique": true })
{
 "createdCollectionAutomatically" : false,
 "numIndexesBefore" : 1,
 "numIndexesAfter" : 2,
 "ok" : 1
}
```

两者之间扫描文件的数量相差巨大 ~ 1038:

| 方法 | 扫描文件 |
| --- | --- |
| 没有索引 | 有索引 |
| 1039 | 1 |

## **4\. 使用PM2集群模式优化API**

PM2是为Node.js应用程序设计的生产流程管理器。它内置了负载平衡器，允许应用程序在不修改代码的情况下，作为多个进程运行。

使用PM2时的应用停机时间几乎为零。总体来说，PM2确实可以提升API性能和并发性。

在生产环境中部署代码并运行以下命令以查看PM2集群如何在所有可用CPU上进行扩展：

```js
pm2 start  app.js -i 0
```

## **5\. 减少TTFB(第一字节时间)**

第一字节时间是一种测量方式，用作表示web服务器或者其他网络资源的响应时间。TTFB测量从用户或客户发出HTTP请求到客户的浏览器收到页面的第一个字节的时间。

所有用户访问浏览器的同一页面加载速度不可能在100毫秒之内，这仅仅是因为服务器和用户之间的物理距离。

我们可以通过使用CDN和全球本地数据中心缓存内容来减少第一个字节的时间。这有助于用户以最小的延迟访问内容。你可以从Cloudflare提供的CDN解决方案开始着手。

## **6\. 使用带日志的错误脚本**

监视API是否正常工作最好的办法是记录行为，于是记录日志就派上用场。

一个常见的办法是将记录打印在控制台上 (使用`console.log()`)。

比`console.log()`更高效的方法是使用Morgan、Buyan和Winston。我将在这里以Winston为例。

### 如何使用Winston记录 – 功能

-   支持4个可以自由选择的日志等级，如：info、error、verbose、debug、silly 和 warn。
-   支持查询日志
-   简单的分析
-   可以使用相同的类型进行多个transports输出
-   捕获并记录uncaughtException

可以使用以下命令行设置Winston：

```js
npm install winston --save
```

这里是使用Winston记录的基本配置：

```js
const winston = require('winston');

let logger = new winston.Logger({
  transports: [
    new winston.transports.File({
      level: 'verbose',
      timestamp: new Date(),
      filename: 'filelog-verbose.log',
      json: false,
    }),
    new winston.transports.File({
      level: 'error',
      timestamp: new Date(),
      filename: 'filelog-error.log',
      json: false,
    })
  ]
});

logger.stream = {
  write: function(message, encoding) {
    logger.info(message);
  }
};
```

## **7\. 使用HTTP/2而不是HTTP**

除了上述使用的这些技巧，我们还可以使用HTTP/2而不是HTTP，因为它具备以下优势：

-   多路复用
-   头部压缩
-   服务器推送
-   二进制格式

它专注提高性能，并解决HTTP的问题。它使网页浏览更快、更容易，并且消耗更少的带宽。

## **8\. 并行任务**

使用[async.js](https://caolan.github.io/async/v3/)来运行任务。并行任务对API的性能有很大改善，它减少了延迟并最大限度地减少了阻塞操作。
  
并行意味着同时运行多个任务。当你并行任务的时候，不需要控制程序的执行顺序。

以下是一个数组异步并行的简单例子：

```js
const async = require("async");
// 使用对象而不是数组
async.parallel({
  task1: function(callback) {
    setTimeout(function() {
      console.log('Task One');
      callback(null, 1);
    }, 200);
  },
  task2: function(callback) {
    setTimeout(function() {
      console.log('Task Two');
      callback(null, 2);
    }, 100);
    }
}, function(err, results) {
  console.log(results);
  // 结果相当于: {task2: 2, task1: 1}
});
```

在以上例子中，我们使用了[async.js](https://caolan.github.io/async/v3/)以异步的形式执行了两个任务。task 1需要200毫秒完成，但是 task 2不需要等待task 1完成后再执行 – 它在设定的100毫米后执行。

并行任务对API的性能有很大的影响。它减少了延迟并最大限度地减少了阻塞操作。

## **9\. 使用Redis缓存应用**

Redis是Memcached的高级版本。它通过在服务器的主内存中存储和检索数据来优化API响应时间。它提高了数据库查询的性能，也减少了访问延迟。

在下面的代码片段中，我们分别调用了不使用Redis和使用Redis的API，并比较了响应时间。

响应时间差异巨大~ 899.37毫秒:

| 方法 | 响应时间 |
| --- | --- |
| 不使用Redis | 使用 Redis |
| 900ms | 0.621 |

以下是不使用Redis的Node:

```js
'use strict';

//定义需要的所有依赖项
const express = require('express');
const responseTime = require('response-time')
const axios = require('axios');

//加载Express框架
var app = express();

//创建在响应头中添加X-Response-Time的中间件
app.use(responseTime());

const getBook = (req, res) => {
  let isbn = req.query.isbn;
  let url = `https://www.googleapis.com/books/v1/volumes?q=isbn:${isbn}`;
  axios.get(url)
    .then(response => {
      let book = response.data.items
      res.send(book);
    })
    .catch(err => {
      res.send('The book you are looking for is not found !!!');
    });
};

app.get('/book', getBook);

app.listen(3000, function() {
  console.log('Your node is running on port 3000 !!!')
});
```

以下是使用Redis的Node:

```js
'use strict';

//定义需要的所有依赖项
const express = require('express');
const responseTime = require('response-time')
const axios = require('axios');
const redis = require('redis');
const client = redis.createClient();

//加载Express框架
var app = express();

//创建在响应头中添加X-Response-Time的中间件
app.use(responseTime());

const getBook = (req, res) => {
  let isbn = req.query.isbn;
  let url = `https://www.googleapis.com/books/v1/volumes?q=isbn:${isbn}`;
  return axios.get(url)
    .then(response => {
      let book = response.data.items;
      //设置string-key:缓存中的isbn。以及缓存的内容: title
      // 设置缓存的过期时间为1个小时 (60分钟)
      client.setex(isbn, 3600, JSON.stringify(book));

      res.send(book);
    })
    .catch(err => {
      res.send('The book you are looking for is not found !!!');
    });
};

const getCache = (req, res) => {
  let isbn = req.query.isbn;
  //对照服务器的redis检查缓存数据
  client.get(isbn, (err, result) => {
    if (result) {
      res.send(result);
    } else {
      getBook(req, res);
    }
  });
}
app.get('/book', getCache);

app.listen(3000, function() {
  console.log('Your node is running on port 3000 !!!')
)};
```

## 总结

在本指南中，我们了解了如何优化Node.js API的响应时间。

JavaScript重度依赖函数，因此，使用异步函数可以使脚本运行得更快并且不阻塞。

除此之外，我们还可以使用缓存记忆(Redis)、数据库索引、TTFB和PM2集群来提高响应速度。

最后请记住，注意路由的安全性并尽可能优化路由也很重要。我们不能为了提高API响应速度而妥协掉安全性。因此，在Node.js中构建优化的API时，应该保留所有标准安全检查。

你可以在[LinkedIn](https://www.linkedin.com/in/kadeniyi/)上联系我。

后会有期!
