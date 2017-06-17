title: 判断对象内容是否为空
date: 2015-2-1 11:44:12
tags:
- javascript 
categories: 编程语言
---
如题所说的判断对象为空指的是其里面是否有其属性和对应的值，而不是像以前普通的判断非空方法，如下：

```javascript
var x;
if   (typeOf(x)   ==   "undefined")
if   (typeOf(x)   !=   "object")
if	 (!x)

//亦或是下面这种
if(x === null)
if(x === '')

```

现在的情况是x是一个已经定义了的对象，但里面没有任何属性，所以如果再向上面的那些判断方式的话，是会出问题的，可以参考下面的方式：

```javascript
var voteObject = {"ds":1};
//var voteObject = {};
var hasProp;

for (var prop in voteObject){
    hasProp = true;
    break;
}
if (hasProp){
    console.log('对象里有属性');
}else{
    console.error('对象里面为空');
}

```
<!-- more -->

再补充一个题外点，就是如何判断对象是否为数组，具体方法如下：

```javascript
var object = [];
 if(object instanceof  Array)

```

