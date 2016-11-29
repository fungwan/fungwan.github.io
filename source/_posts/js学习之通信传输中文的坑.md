title: 踩中文的坑 
date: 2016-6-15 10:44:12
tags:
- javascript 
categories: javascript
---
如标题所示，这次笔记一个工作中遇到的小坑，虽说小却楞是花了我几大小时的时间。当时的背景是这样的，我需要在客户端这边发送一个http的POST请求，然后服务器根据请求执行相应的db操作。代码类似下面这样子：

```javascript
var http = require('http');

var bookBody = {
    "title": "黑客与画家",
    "price":9.89
}

var bookBodyString = JSON.stringify(bookBody);
var options = {
    host: 'localhost',
    port:'3000',
    path: '/books',
    method: 'POST',
    headers: {
        'Accept': '/',
        'Content-Type':'application/json',
        'Content-Length': bookBodyString.length
    }
};

var req = http.request(options, function(res) {
    res.setEncoding('utf8');
    res.on('data', function(data) {
        console.log(data);
    });

    res.on('err',function(data){
        console.log(data);
    });
});


req.write(bookBodyString);
req.end();`
```

结果代码执行过后，对方的服务器给挂掉了，当然我这边也就没有响应到正确的数据，服务器报出了语法的错，类似‘无法解析的结尾’。当时我就在想，这部分的请求代码是我直接拷过来的（话说直接拷代码是非常‘危险’的动作），拷的这部分代码在之前是成功执行的，只是我这边稍微改了一下，难不成是因为这次改动的原因，然后我回滚代码，让它几乎保持了与拷贝之前一致性，接着再执行，晕~依然崩溃。仔细对比了代码，发现剩下不同的就只有新的代码里面消息体包含了中文。于是换成英文再执行,OK。那依这样看是不是就是该服务端的问题呢，即对方在接收数据时没有做正确的处理，或者没有做中文编码之类的处理。于是，我把这个问题反馈给了作者，[这篇](https://github.com/TossShinHwa/node-odata/issues/57 "色拉油啊油")的6楼是我的反馈记录。大家可以看看。

那从上面的链接我们可以牢记一个点是，在请求的头部，content-length它表示的是所在消息体的字节数而不是字符个数，明白这个道理的话就释然了

