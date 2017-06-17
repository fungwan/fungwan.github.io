title: node-schedule的使用说明
date: 2014-10-11 16:44:12
tags:
- node.js
- exports
categories: 后端开发
---

这篇笔记内容很简单，就只是记录一下node-schedule的使用方法，特别是在初始化一个定时任务时，需要传递的参数的写法含义。

先看一段代码：

```javascript
var schedule = require('node-schedule');

function createScheduleCron(){
    schedule.scheduleJob('30 * * * * *', function(){
        console.log('excute scheduleCronstyle:' + new Date());
    }); 
}

createScheduleCron();
```
<!-- more -->

看看构造函数中通配符的含义吧：

>*  *  *  *  *  *
>┬ ┬ ┬ ┬ ┬ ┬
>│ │ │ │ │  |
>│ │ │ │ │ └ day of week (0 - 7) (0 or 7 is Sun)
>│ │ │ │ └───── month (1 - 12)
>│ │ │ └────────── day of month (1 - 31)
>│ │ └─────────────── hour (0 - 23)
>│ └──────────────────── minute (0 - 59)
>└───────────────────────── second (0 - 59, OPTIONAL)

**提醒**，这里要特别注意对应每位上的数字区间，不然定时任务会启动失败。

 1. 6个占位符从左到右分别代表：秒、分、时、日、月、周几
 2. '*'表示通配符，匹配任意，当秒是'*'时，表示任意秒数都触发，其它类推

----------

举个例子：
>每分钟的第30秒触发： '30 * * * * *'
>每小时的1分30秒触发 ：'30 1 * * * *'
>每天的凌晨1点1分30秒触发 ：'30 1 1 * * *'
>每月的1日1点1分30秒触发 ：'30 1 1 1 * *'
>2016年的1月1日1点1分30秒触发 ：'30 1 1 1 2016 *'
>每周1的1点1分30秒触发 ：'30 1 1 * * 1'

看完是不是有点豁然开朗，最后特别鸣谢这篇博文举的[例子][1]。
  [1]: http://www.cnblogs.com/zhongweiv/p/node_schedule.html