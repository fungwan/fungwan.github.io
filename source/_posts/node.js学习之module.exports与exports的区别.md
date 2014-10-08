title: node.js中module.exports与exports的区别
date: 2014-09-30 12:44:12
tags:
- node.js
- exports
categories: node.js
toc: false
---

我们在阅读它人或开源的node.js代码时，会发现在对模块导出的写法上会出现下面几种情况，而关于导出的方法整理，可以参见我这篇文章[node.js中exports的用法整理](http://fungwan.me/2014/09/29/node.js%E5%AD%A6%E4%B9%A0%E4%B9%8Bexports/)。现在，我们以导出字符串对象为例：


在module.js文件中，有如下写法：

* 1.exports.get_name = 'fungwan by exports ';
* 2.module.exports.get_name = 'fungwan by modules';
* 3.module.exports = 'fungwan by modules';

通过对比发现，这里较之前多了module.exports，那么它跟exports的导出有什么关系呢？查阅资料即可知晓，其实exports就是module.exports对象的引用，本质上是相等的。我们可以通过这句代码证实：

```javascript
console.log(module.exports === exports);
```

它打印出来是true。不过我们真正要require出来的是只是module.exports这个对象！

等等，这里又提到了引用，相信熟悉C++或Java的人应该都对引用或有深刻的认识，即我们对引用的对象进行了属性的修改，那么被引用的对象的属性也会被修改。这就是为什么我们在module.js仅对exports.some_words属性进行修改，而在真正导出的module.exports的对象中，我们在外面（另一个需要require的文件）也可拿到some_words这个字符串对象值。

这里我们新建一个main.cpp文件，用C++代码来说明一下上述观点：


```c

//新建一个类C_require，待会儿会实例化一个名为module_exports的对象
class C_require
{
public:
    C_require(void);
    virtual ~C_require(void);

public:
	void modifyName(string& name);
	string getName();
private:
	string m_name;

};
C_require::C_require(){
	m_name = "冯云";
}

C_require::~C_require(){

}

void C_require::modifyName(string& after_name){
	m_name = after_name;
}

string C_require::getName(){
	m_return name;
}

```

接着我们在入口函数里面，去实例化一个名为module_exports的对象，就类似于我们真正require的是module_exports。


```c

int _tmain(int argc, _TCHAR* argv[])
{
	C_require module_exports;
	string name = module_exports.getName();
	cout<<"我是module_exports，中文名叫："<<name<<endl;

	C_require& exports = module_exports;//exports拥有module_exports一样的属性
	cout<<"我是module_exports的引用exports，中文名做："<<name<<endl;

	//首先，我们通过对引用的对象进行其属性name的修改
	string english_name = "andy.fung";
	exports.modifyName(english_name);//当exports去修改属性时，module_exports的属性也会变，因为他们都指向的一块内存区域

	//接着，我们再看被引用的对象module_exports中name属性值的已经改变
	string after_name = module_exports.getName();
	cout<<"我是module_exports，我的名字被exports改成了英文，叫做："<<after_name<<endl;

	return 0;
}

```

但是，当我们在一个类似于module.js的模块文件中，同时有exports和module.exports的导出时，后者是会覆盖前者的导出，所以一般在模块文件中只用其中一种方式导出。

##需要注意的一点
我们在导出的写法上有一点是要避免，因为它是错误的，比如在对字符串对象的导入：

```javascript
exports = 'fungwan by exports ';//error
```

这种导出的方式其实是给exports重新赋值了（内存的指向位置发生了变化），即已不是module.exports的引用，exports的修改影响不到module.exports。所以当我们在外部用require时会发现导出的是一个空{}。还是拿上面C++的例子说明一下，我们继续在main函数里面，重新实例化一个C_require对象，然后用exports指向新的这个对象：

```c

int _tmain(int argc, _TCHAR* argv[])
{
	//现将exports重新赋值，即指向了另一个内存区域
	C_require pRequireOther;
	string english_name2 = "jacky.fung";
	exports.modifyName(english_name2);
	exports = pRequireOther;
	string otherName = exports.getName();
	cout<<"我是exports，我的名字变成了："<<otherName<<endl;//新名jacky.fung
	cout<<"我是module_exports，我的名字变成了："<<module_exports.getName()<<endl;//还是andy.fung

	return 0;
}

```

针对上面的错误的导出方式有一个解决办法：
```javascript
exports = module.exports = 'fungwan by exports ';
```

但是如果像这样，还不如直接就只写成`module.exports = 'fungwan by exports'`;

最后，再来说说根据不同的导出方式，我们在require的时候需要注意些什么？关键还是要理解属性跟对象的关系。还是以代码的方式，下面我都以module.exports导出，可以跟之前的这篇[node.js中exports的用法整理](http://fungwan.me/2014/09/29/node.js%E5%AD%A6%E4%B9%A0%E4%B9%8Bexports/)的代码进行对比：

导出一个字符串
```javascript
module.exports = 'string by modules';
```

导出一个类
```javascript
function helper() {
    var number = '';
    this.do = function () {
        console.log('I can help you!');
    }
};

helper.prototype.giveMoney = function(){
    console.log('I have lots of cash!');
}

module.exports = helper;
```

那么在require的时候，我们的方式是：

require一个字符串
```javascript
var lib = require('./module.js');
console.log(lib);
```


require一个类对象
```javascript
var lib = require('./module.js');
var c_help =  new lib();
c_help.giveMoney();
```
