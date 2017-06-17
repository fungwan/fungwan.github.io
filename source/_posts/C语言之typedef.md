title: 温习typedef
date: 2014-10-16 19:44:12
tags:
- javascript
categories: 编程语言
---
typedef是c语言里面基础的语法知识，作用有点跟#define类似，但是如果它与其他结构搅在一起，还是会让你在风中凌乱。此次笔记就不说简单的typedef基本类型，只重点说下typedef结构体~<!-- more -->

还是先回顾一下结构体的声明和定义吧：
```javascript
struct singer 
{  
	int no;  
	int score;
};
```

上面是一个名为singer的结构体，如果用它来定义一个结构体变量的话，可以这样做：
```javascript
struct singer andyLau;
```

需要注意的是，这样的定义是不能省略struct关键字的，否则编译都不通过！而如果要定义一个指向结构体变量的指针，可以像定义其他基本数据类型指针一样的做法：

```javascript
struct singer* andyLau;
```

可能很多人都会觉得每次都要重复写struct会很麻烦，所以就引入了typedef,像下面这样写，指向结构体的指针也是一样：

```javascript
typedef struct _singer 
{  
	int no;  
	int score;
}Singer,* Ptr_Singer;//把*后面空上一格会好理解一点
```

我们可以这么来看，typedef把原本要这么写的定义变量，改成了下面这种：

```javascript
Singer andyLay;//等同于 struct singer andyLau;
Ptr_Singer andyLau;//等同于 struct singer* andyLau
```

更深层次一点就是，如果是指向这个结构体指针的指针又该如何表示呢？原本是：
```javascript
Ptr_Singer* pointAndyLau;
```

我们用先typedef下，然后再定义可以变成：
```javascript
typedef Ptr_Singer* ToPtr_Singer;
ToPtr_Singer pointAndyLau;
```

下面写一个完整的例子：

```javascript
Singer andyLau = {10, 98};
Ptr_Singer andyLauCopy = &andyLau; // 完全等价于Singer* andyLauCopy = &andyLau;
ToPtr_Singer toAndyLau = &andyLauCopy; // 完全等价于Singer** toAndyLau = &andyLauCopy;

Ptr_Singer p=(Ptr_Singer)malloc(sizeof(Singer));//引申一下初始化一个结构体指针
Ptr_Singer p1 = new Singer;//c++可以这样
```

如果是结构体里面又存放了结构体，就像链表这样的数据结构，参考两篇文章 [ typedef使用大全](http://blog.chinaunix.net/uid-20659461-id-1905089.html/ "") ， [ 关于typedef的用法总结](http://www.cnblogs.com/csyisong/archive/2009/01/09/1372363.html "")

```javascript
typedef struct _TTS1{
    typedef struct ITTS1 {
        int x, y;
    } iner;
    iner i;
    int x, y;
} TTS1;

//////////////////////(对比下面这个)

typedef struct node{
	Elemtype elem;
	struct node* next;
}ListNode,*ListNodePtr;

```

我们其实还是可以一样可以结构体内部的结构体，像这样：

```javascript
typedef TTS1::ITTS1 ITS1;
```

操练起来，把这个链表初始化用下：


```javascript
	TTS1 itts1 = {{110, 220}, 10, 20};
    Its1* rits1 = &itts1.i;
    ITS1* &its1 = rits1; // 等价于 TTS1::ITTS1 *its1 = &(itts1.i);
```

有没有很昏的感觉，哈哈！