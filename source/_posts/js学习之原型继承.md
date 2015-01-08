title: 理解js中的原型继承
date: 2015-1-5 12:44:12
tags:
- 面向对象 
categories: javascript
---

为了搞清在js中如何实现继承，也是对之前这篇博文[js中prototype用法](
http://fungwan.me/2014/12/24/js%E5%AD%A6%E4%B9%A0%E4%B9%8Bprototype%E7%94%A8%E6%B3%95/ "冯云的博客")的补充，我主要参考了下面两篇文章：
>伯乐在线的[JavaScript原型和继承](http://blog.jobbole.com/19795/)
>
>阮一峰的[构造函数的继承](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html)
	
我是两篇对比着看的，刚开始看的时候脑袋都快被绕晕了，或许是自己对于js的高级特性不熟悉，亦或是js本身语言的强大灵活性与传统的编译性语言区别太大的原因导致，所以此次的学习笔记决定‘从简’，为了不搞混淆，只把我觉得易懂的部分给抽出来，方便以后查阅。<!-- more -->

介绍继承之前，必须先理解一个概念，就是定义/创建一个函数的过程中做了哪些事，这样的话对后面理顺原型链是有帮助作用的，举个上面参考链接的例子和配图：

```javascript

function Person() {};

```

像这样创建一个空函数，js解析为以下三步：

1. 创建一个Object对象（有constructor属性及[[Prototype]]属性）;
2. 创建一个函数（有name、prototype属性），再通过prototype属性引用刚才创建的对象;
3. 创建变量A，同时把函数的 引用 赋值给变量A

如图可以表示为：
![](/img/prototype_1.png "Optional title")


那我们用这个函数去实例化一个对象时，js解析又是怎样呢？比如：

```javascript

var people = new Person();

```
实例化出来的对象，js解析也分为下面三步：

1. 新建一个对象并赋值给变量people：var people = {};
2. 把这个对象的[[Prototype]]属性指向函数Person的原型对象：people.[[Prototype]] = Person.prototype
3. 调用函数Person，同时把this指向刚创建的对象people，对对象进行*初始化*：Person.apply(people,arguments)

如图可以表示为：
![](/img/prototype_2.png "Optional title")

那么现在来引入继承，首先我们把最开始的函数Person改造一下，变成：

```javascript

function Person(name) {
	this.data = [1,2,3];
	this.name = name;
};

//添加原型方法
Person.prototype.sayHello = function(){
	console.log('大家好');
}
```

接着声明一个函数Woman,让它继承于Person，先用prototype模式写，如下：

```javascript

function Woman(name) {};
Woman.prototype = new Person(name);

```

解释一下为什么这样写，最后一行的意思是，将Woman的prototype对象指向一个Person的实例,相当于完全删除了Woman的prototype对象原先的值，然后赋予一个新值。如图可以表示：

![](/img/prototype_3.png "Optional title")

但是由于上面最后一行的改变直接导致了Woman的prototype对象里面没有了constructor属性，只能从原型链找到Person.prototype读出constructor:Person，那么以后靠Woman实例化出来的对象就会造成继承链的紊乱，所以这里要改一下代码，将Woman.prototype对象的constructor值改为Woman。如：

```javascript

function Woman(name) {};
Woman.prototype = new Person(name);
Woman.prototype.constructor = Woman;

```
有点绕，再用图表示吧，如下：
![](/img/prototype_4.png "Optional title")


基类和派生类写好后，我们可以试着去用Woman实例化一个对象，访问一下基类提供的方法，如下：

```javascript

var angela = new Woman('杨颖');
angela.sayHello();

```

从运行结果来看可知这是可行的，暂时没有问题。但是，如果我们继续改写这个代码，像这样：

```javascript

var angela = new Woman('杨颖');
angela.sayHello();
angela.data.push(4);//改变prototype的data数组

//再用Woman实例化一个对象
var baby = new Woman('凑莉久');
//再用baby去访问数组data,发现其值已改变
console.log(baby.data);//[1,2,3,4]

```
画个图表示下：
![](/img/prototype_5.png "Optional title")

这一次从结果上(data的输出)来看，显然是不符合要求的，其实我们想要的只是原型链，那么我们用中间者的方式修改：

```javascript

var F = function(){};
F.prototype = Person.prototype;
Woman.prototype = new F();
Woman.prototype.constructor = Woman;

```
也是用图来表示一下：
![](/img/prototype_6.png "Optional title")

将上面的方法，可以封装成一个函数:

```javascript

function extend(Child, Parent) {

　　　var F = function(){};
　　　F.prototype = Parent.prototype;
　　　Child.prototype = new F();
　　　Child.prototype.constructor = Child;
}

```

我们使用的时候就可以直接这样：

```javascript

extend(Woman,Person);
var baby = new Woman("杨颖");
console.log(baby.data);

```

这篇文章可能总结的不太全（比如还可以用拷贝实现继承）或者还有错误的地方，待以后技术扎实后再回看吧~