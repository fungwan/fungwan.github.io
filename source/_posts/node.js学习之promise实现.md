title: node中异步回调解决方案之promise
date: 2014-11-11 17:44:13
tags:
- node.js
- promise
categories: 后端开发
---

以前在其他开发平台写码习惯了同步的写法，现在切换到node.js里面的异步特性，还是有一点的不习惯，起初还不怎么觉得，但当遇到数据库的操作，并且需要其返回的数据作下一步的逻辑时，问题就出来了，后面的逻辑都会写在前一个异步的回调里面，如此炮制，最后的结果就是一个金字塔似的回调噩梦就产生了。

那么如何解决这样的噩梦呢？其实不管是网上还是其他渠道的资料上都有其针对异步的解决方案，而我是在看了张丹老师的博文后，决定尝试使用asyn库，并且应用在了实际的项目中来，效果不错。而今天我要笔记的是另外一种异步的解决方案-promise。<!-- more -->

什么是promise？根据官方定义：Promise是一个对象，它通常代表一个在未来可能完成的异步操作。这个操作可能成功也可能失败，所以它一般有3个状态：Pending，Fulfilled，Rejected。分别代表未完成、成功完成和操作失败。一旦Promise对象的状态从Pending变成Fulfilled或者Rejected任意一个，它的状态都没有办法再被改变。

一个Promise对象通常会有一个then方法，这个方法让我们可以去操作未来可能成功后返回的值或者是失败的原因。这个then方法是这样子的：

    promise.then(onFulfilled, onRejected)


从上面的方法可以看出，then方法接受两个参数，它们通常是两个函数，一个是用来处理操作成功后的结果的，另一个是用来处理操作失败后的原因的，这两个函数的第一个参数分别是成功后的结果和失败的原因。如果传给then方法的不是一个函数，那么这个参数会被忽略。

而这个方法的返回值也是一个promise对象，所以我们又可以将其结果再次的调用then方法以达到流程控制的效果。那对于其中参数的传递和错误处理等等，官方有做了如下定义：

1. onFulfilled或者onRejected函数的返回值不是Promise对象，则该值将会作为下一个then方法中onFulfilled的第一个参数，如果返回值是一个Promise对象，那么then方法的返回值就是该Promise对象；

2. onFulfilled或者onRejected函数中如果有异常抛出，则该then方法的返回的Promise对象状态转为Rejected，如果该Promise对象调用then，则Error对象会作为onRejected函数的第一个参数；

3. 如果Promise状态变为Fulfilled而在then方法中没有提供onFulfilled函数，则then方法返回的Promise对象状态变为Fulfilled且成功的结果为上一个Promise的结果，Rejected同理。


再次说明一下，上面只是介绍的promise的规范定义，我们如果想应用它的话，还需要找到实现其这样规范的库。这里我们将以Q模块举例。（现在的es6标准好像已经将promise纳入了进去，提供了原生的API支持，好事，好事呀。。。）

多数Promise的实现在Promise的创建上大同小异，通过创建一个具有promise属性的defer对象，如果成功获取到值则调用defer.resolve(value)，如果失败，则调用defer.reject(reason)，最后返回defer的promise属性即可。

先举一个以前不用异步回调的解决方案的例子：

```javascript
step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // Do something with value4
            });
        });
    });
});
```

是不是很痛苦的样子？我们用q模块来将上面的例子作一次改写：

```javascript
var Q = require("q");

var step1 = function (data) {
    var deferred = Q.defer();
    deferred.resolve(data+" fun1");
    //deferred.reject('cuol');
    return deferred.promise;
};

var step2 = function (data) {
    var deferred = Q.defer();
    deferred.resolve(data+" fun2");
    return deferred.promise;
};

var step3 = function (data) {
    var deferred = Q.defer();
    deferred.resolve(data+" fun3");
    return deferred.promise;
};

var step4 = function (data) {
    console.log('错误原因' + data);
    return 'boduo';
};

step1(data)
        .then(step2,step4)
        .then(step3)
        .catch(function(ex) {
            console.log(ex);
        });
        .done(function(data){//等同于finally
            cb(null,data);//ok 获得的最终数据为 --->"test fun1 fun2 fun3"
        },function(err){
            cb(err);//failed
        });
        .fail(function(data){
            console.log('error');
        });

```

在上面的代码中，then方法既接受OnFulfilled，又接受OnRejected，如果一系列异步方法只要始终是成功返回值的，那么代码就会瀑布式的向下运行，如果其中任意一个异步方法失败或者发生异常，那么根据CommonJs的Promise规范，将执行fail方法或者catch中的function。q还提供了finally方法，从字面上也很好理解，就是不论resolve还是reject，最终都会执行finally中的function。

对于是全部并发的执行上面的step呢？q也提供了all的方法，如下：

```javascript
   q.all([step1, step2, step3]).spread(function(val0, val1, val2){
         console.log(arguments);////获得的参数为('test1 fun1', 'test2 fun2', 'test3 fun3' )
         }).then(function(){
            console.log("done");
         }).catch(function(err){
            console.log(err);
         });
```

这篇笔记是对Promises的一个简单认识。用粗浅的代码感受一下它的规范实现，还有它在封装具有异步方法的函数时的优势提现，那么真正理解的话还是要在项目中多加以运用才行噢。