title: 谈谈js中的作用域
date: 2015-6-22 12:44:12
tags:
- javascript 
categories: 编程语言
---
如果有过其他高级语言‘块级作用域’的认知，那么再接触js的函数作用域后就会大跌眼镜。个人认为应该在这几个方面着重关注一下：作用域、作用域链、变量提升。

先看作用域，js是没有块作用域的说法的（不过现在ES6的let关键字已经支持了），用网络上其他的例子作说明：

```javascript
var name="global";
if(true){
    var name="local";
    console.log(name)//local
}
console.log(name);//local
```

上面的输出都为local,如果不是函数作用域而是块作用域的话，输出应该依次是local和global。所以，以上代码证明了开始的观点。这里一定要注意**函数作用域**这个关键点哦！<!-- more -->

再来看一个作用域链的例子，

```javascript
var num = 10;
var func1 = function() {
 var num = 20;
 var func2 = function() {
  var num = 30;
  console.log(num);
 };
 func2();
};
var func2 = function() {
 var num = 20;
 var func3 = function() {
  console.log(num);
 };
 func3();
};
func1();
func2();
```

反正总的思想就是，查看欲访问的对象属于当前哪一个作用域，如果找不到则向上级逐级查找，直到查找最顶级作用域，如果还没定义就返回undefined。比如上面的例子，运行func2()的时候，我们程序为了打印num,发现在当前函数作用域里面没有发现num的定义，于是往上级查找，在func2的函数作用域里面有num和func3（函数也是对象哦，别忘了）.那么这里自然有num的定义，即它找到了，所以输出20，同理运行func1(),就会输出已经在当前作用域找到的num为30。

最后我们来说说变量提升，这个就更变态了，稍不注意就会弄错，还是以上面的例子为例，只不过改一个地方，缩减版：

```javascript
var num = 10;

var func2 = function() {

 var func3 = function() {
  console.log(num);
 };

 func3();

 var num = 20;
};

func2();

```

猜猜运行func2()会是什么结果呢？如果按照上面的思想你答20那就错了或者说只离成功只差一半啦，还是先按作用域链的思想进行分析，我们欲打印num，然后在func2所在的函数域里面找到了num，但是这里要**注意**，此时的num并没有赋值，因为代码是从上往下运行的。是不是很坑。所以打印出来是undefined。这种现象就叫做变量提升，所以，你将var num = 2放到func3()的前面，结果是不是又不一样呢?哈哈。

以上为自己学习js函数作用域的一些愚见、小结，最后我感觉这逼就是一坑啊作用域特性，不知当时作者为什么要设计成与其他高级语言背道而驰的作用域概念。所以以后写码的时候特别是node后端，要用到的变量提前写到最前面在一个作用域里面，当然用ES6也是个不错的选择！

