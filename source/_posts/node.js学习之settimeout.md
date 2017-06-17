title: node.js中的定时器setInterval
date: 2016-07-21 17:44:12
tags:
- node.js
categories: 后端开发
---
定时器相信大家都用过，而且才开始接触时，老是与线程混淆(特别是自己原来写MFC程序时)，这里有一篇文章对这两个概念作了很好的对比【[定时器和多线程对比](http://blog.csdn.net/wangweitingaabbcc/article/details/6723606)】。这里我只是在node.js层面作个补充。<!-- more -->

为什么会有这个想法，因为之前在作项目的codeReview时，发现我的代码里面有一个维护在线用户的列表的数组，定时器会实时检测它是否有掉线的情况，并做相应处理，而我程序的主线程（这里暂叫吧，习惯了）也在操作这个数组，害怕他们会把数组的数据‘搞烂’，作了不必要的判断。当然这全是因为当时对定时器和线程概念不太清晰，殊不知他们其实都在一个线程里面，是串行的，相互间不会影响。不会像从前写MFC时候，为避免类似错误，子线程或者定时器里面操作同一份数据时要加上互斥锁！

举一个人家的[例子](http://www.admin10000.com/document/4196.html)，代码如下：

```javascript

var start = Date.now();//获取当前时间戳
setInterval(function () {
    console.log(Date.now() - start);
    for (var i = 0; i < 1000000000; i++){//执行长循环
    }
}, 1000);
setInterval(function () {
    console.log(Date.now() - start);
}, 2000);


```

先来看看最后的打印结果：

```javascript
1000
3738
```

对于我们期望2秒后执行的setInterval函数其实经过了3738毫秒之后才执行，换而言之，因为执行了一个很长的for循环，所以我们整个Node.js主线程被阻塞了，如果在我们处理100个用户请求中，其中第一个有需要这样大量的计算，那么其余99个就都会被延迟执行。对于setTimeout的行为，可以参考《深入浅出node.js》P61的流程图，相信会有更深的理解。

所以虽然Node.js可以处理数以千记的并发，但是一个Node.js进程在某一时刻其实只是在处理一个请求。

