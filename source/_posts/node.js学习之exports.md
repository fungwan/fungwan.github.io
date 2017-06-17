title: node.js中exports的用法整理
date: 2014-09-29 18:44:12
tags:
- node.js
- exports
categories: 后端开发
---

相信很多跟我一样是node.js新手的同学对模块的导出或多或少有些混淆，现在我把exports对象的方式整理一下，其中包含了模块中需要导出的string对象、json对象、函数、类对象。

我们将需要导出的模块就命名为module.js，通过下面的代码导出需要的对象：


```javascript
exports.some_words = 'this is string';//导出一个字符串

exports.some_fun = function() {//导出一个函数
    console.log('this is function');
};

exports.jsonObj = {
    name : 'fungwan',
    sex  : 'man',
    run  : function(){
        console.log('love running!');
    }
};

//导出一个类
function helper() {
    var number = '';
    this.do = function () {
        console.log('I can help you!');
    }
}

helper.prototype.giveMoney = function(){
    console.log('I have lots of cash!');
}

exports.c_help = helper;
```
<!-- more -->
那么再新建一文件为调用方，暂命名为main.js，通过下面的代码进行调用：

```javascript
var lib = require('./module.js');

var some_words = lib.some_words;
console.log(some_words);

lib.some_fun();

var myFeature = lib.jsonObj;
console.log(myFeature.name);
myFeature.run();

var c_help =  new lib.c_help();
c_help.do();
c_help.giveMoney();
```

好了，我们看一下运行结果：


``` bash
this is string
this is function
fungwan
love running!
I can help you!
I have lots of cash!
```