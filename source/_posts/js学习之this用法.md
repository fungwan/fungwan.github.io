title: 学习js中的this用法
date: 2014-12-23 09:44:12
tags:
- javascript
categories: 编程语言
toc: false
---

通过这几个月在node.js上的工作，发现js其实并不是那么简单，而且感觉上越来越深，比如现在要笔记的一个坑——this,因为之前用过C++对this有一定的了解，其就是指向当前对象，而javascript中的this，因其弱语言的灵活性，它被赋予了更多的作用和使用范围。由于其运行期绑定的特性，它可以是全局对象、当前对象或者任意对象，这完全取决于函数的调用方式，下面根据网上的文章作了一个归纳，方便日后温习。

一般而言，在Javascript中，this指向函数执行时的当前对象，该关键字在Javascript中的执行环境，而非声明环境有关。
##1.作为单纯的函数调用
```javascript
function modifyName(name) {
    this.name = name;
}
modifyName('fengyun');
console.log(name);
```
<!-- more -->
当函数直接被调用，此时this绑定到了作用域更大的全局对象！比如这里多出的一个全局对象name,打印出来的结果为fengyun。但是注意，如果前提有了全局对象，结果值还是之前的全局对象，比如下面：
```javascript
var name = 'liuchunping';
function modifyName(name) {
    this.name = name;
}
modifyName('fengyun');
console.log(name);//liuchunping
```
##2.内部函数
从上述例子我们知道当作为函数单独调用的时候，this绑定到了全局对象，那么如果这个单纯函数被声明到了另一个函数里面，即成为内部函数，如果处理不好会引发一个不希望的错误。比如，我想修改person的name属性，那么通过以下方式非但没有修改成功，而且还多了一个全局对象name。
```javascript
var person = {
name : "foocoder",
setName : function(sth){
    var modifyName = function (name) {
        this.name = name;
    }
    modifyName(sth);
  }
}
person.setName("fengyun");
console.log(person.name);//foocoder
console.log(name);//fengyun

```
`tips`:这属于JavaScript的设计缺陷，正确的设计方式是内部函数的this应该绑定到其外层函数对应的对象上，为了规避这一设计缺陷，约定俗成，该变量一般被命名为that或者_self。修改方法如下：
```javascript
var person = {
name : "foocoder",
setname : function(sth){
    var _self = this;
    var modifyname = function (name) {
        _self.name = name;
    }
    modifyname(sth);
  }
}
person.setname("fengyun");
console.log(person.name);//fengyun
//console.log(name);//error报错，name为undefined，说明没有挂载到全局对象上

```
##3.作为对象调用
在JavaScript中，函数也是对象，因此函数可以作为一个对象的属性，此时该函数被称为该对象的方法，在使用这种调用方式时，this 被自然绑定到该对象上。
```javascript
var person = {
name : "foocoder",
setname : function(name){
     this.name = name;
  }
}
person.setname("fengyun");
console.log(person.name);//fengyun
```
##4.作为构造函数调用
JavaScript 并没有类（class）的概念，而是使用基于原型（prototype）的继承方式，它的构造函数很特殊，如果不使用new调用，this的挂载则和普通函数一样。作为又一项约定俗成的准则，构造函数以大写字母开头，提醒调用者使用正确的方式调用。如果调用正确，this绑定到新创建的对象上。
```javascript
function Point(x, y){
    this.x = x;
    this.y = y;

    this.getx = function(){
	console.log(this.x);
    }
 }

Point.prototype.setx = function(value){
    this.x += value;
}
//new的方式,实例化出来的对象各自挂载的x都有了新的值
var p1 = new Point(1,2);
p1.setx(7);
p1.getx();//8
var p2 = new Point(3,4);
p2.setx(9);
p2.getx();//12


//全局对象
//console.log(x);//未找到x，所以为undefined

//普通函数
Point(77,88);
console.log(x);//Point当普通函数时，this挂载了全局对象，x不再为undefined，变为了77

```