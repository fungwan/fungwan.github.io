title: 常用的markdown标记
date: 2014-12-23 17:44:12
tags:
- markdown
categories: markdown

---
##Markdown
平常用markdown的时间不多，所以每次用它来写日志的时候都容易忘记，为此总结一些平常自己会经常用到的标记，方便查看（代码和效果同时附上），此后也会持续更新...

###图片
为致敬女神，图片格式放最前，^_^
```javascript
![Alt text](/img/hexie.jpg "Optional title")
```
![](/img/songdaofeng.jpg "入门老师")

###标题

```javascript
   # 这是 H1 #
   ## 这是 H2 ##
   ### 这是 H3 ######
```
###列表
无序列表使用星号、加号或是减号作为列表标记：
```javascript
*   Red
*   Green
*   Blue
```
效果如下：
*   Red
*   Green
*   Blue

有序列表使用数字接着一个英文句点：
```javascript
1.  Bird
2.  McHale
3.  Parish
```
效果如下：
1.  Bird
2.  McHale
3.  Parish

###链接
链接文字都是用 [方括号] 来标记,想要加上链接的 title 文字，只要在网址后面，用双引号把 title 文字包起来即可~
```javascript
This is [andy.fung](http://fungwan.me/ "冯云的博客") inline link.
```
效果如下：
This is [andy.fung](http://fungwan.me/ "冯云的博客") inline link.

###强调
Markdown 使用星号（*）和底线（_）作为标记强调字词的符号，被 * 或 _ 包围的字词会被转成用 <em> 标签包围，用两个 * 或 _ 包起来的话，则会被转成 <strong>

```javascript
**这就是强调**
```
效果如下：
**这就是强调**

###其他格式，如邮件
Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用尖括号包起来， Markdown 就会自动把它转成链接。

```javascript
<noder.feng@gmail.com>
<http://fungwan.me>
```
效果如下：
><noder.feng@gmail.com>
><http://fungwan.me>