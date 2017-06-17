title: 解惑js中的prototype
date: 2014-12-24 09:44:12
tags:
- javascript
categories: 编程语言
---
平时node中在人家源码里往往会看到prototype，之前百度了下说的是原型方法，也没有细看，今天决定好好把这块知识点学习一下。结合网上的资料，把prototype的定义精简一下：js中每个函数都有一个默认的prototype属性，这个属性是另一个对象（我们称之为‘原型对象’）的引用，换句话说，其实prototype属性就是一个对象。这个原型对象的所有属性和方法，都会被构造函数的实例继承。

说了定义，那我们来看什么时候使用prototype呢？这篇[博文](http://www.cnblogs.com/dolphinX/p/3286177.html "色拉油啊油")对这部分讲的很清楚。按照他的例子举个例，在传统的实例化出来的对象要使用属性或方法时，我们事先是这样构造一个‘类’（为了好理解暂叫类吧）：

```javascript
function Student(){
    this.friends = []; //实例变量
    this.read=function(){ //实例方法
	    var bookName = '代码与艺术';
        console.log('我要读' + bookName);
    }
}

var fengyun = new Student();
fengyun.friends.push('lauchunpung');
fengyun.read();
console.log(fengyun.friends); //['lauchunpung']
        
var andy = new Student();
andy.friends.push('zhuliqian');
andy.read();
console.log(andy.friends); //['zhuliqian']


```
<!-- more -->
从上面的代码可以看到，通过类student，我们实例化了两个对象，fengyun和andy。这两个对象操作属性或方法时，都是独立的互不影响。*所以fengyun中的属性和方法与andy中的属性与方法虽然同名但却不是一个引用，而是对Studen定义的属性和方法的一个拷贝。*所以如果实例化的对象越多，对应的拷贝份数越多。虽然V8有自己的一套内存管理体系，不会像之前C++那样new出来的对象需要自己手动管理delete。但是为了更好的速度和空间，prototype因运而生。

我们这时候用prototype的方式来重写上面的代码：

```javascript
function Student(){
	
}

Student.prototype.friends = []; //原型属性

Student.prototype.read=function(){ //原型方法
	var bookName = '代码与艺术';
	console.log('我要读' + bookName);
}

var fengyun = new Student();
fengyun.friends.push('lauchunpung');
fengyun.read();
console.log(fengyun.friends); //['lauchunpung']
        
var andy = new Student();
andy.friends.push('zhuliqian');
andy.read();
console.log(andy.friends); //['lauchunpung','zhuliqian']

```

从上面的运行结果来看，特别是属性friends,在对象andy的再次操作下，最后也得到了之前fengyun的操作结果，所以数组输出为'liuchunping'和'zhuliqian',由此我们可以看出所有实例的friends属性和read()方法，其实都是同一个内存地址，指向prototype对象，他的属性和方法都属共享状态，另一方面来说，实例的访问它们也会更快（没有拷贝动作）！

上面所说的关于prototype只是初步的一部分，只有‘是什么’，和‘为什么’，为了不在一篇文章里写太多东西而搞复杂，所以后面可能会整理关于prototype的原型继承等更高级的特性~



