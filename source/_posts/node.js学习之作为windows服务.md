title: node.js作为windows服务启动
date: 2015-6-28 11:22:12
tags:
- node.js
categories: node.js
toc: false
---

因为特殊原因，项目里node服务器需要部署在windowsServer上（怀念阿里云啊），很明显之前的forever神器就不能用了。但是回到现实中该怎么在windows上部署呢？不可能叫人家开个cmd窗口吧？怪怪的感觉，而且管理员稍不注意还容易将cmd误操作将其关闭！难不成再叫人家到目标目录，node app.js？

后来到网上搜索找到了一篇文章 [将node.js程序注册为windows服务](http://www.oschina.net/question/12_18694?sort=time "ruan")，然后按照上面的方法照做，也下载了文中提到的两个必备工具：

1. Instrsrv.exe installs and removes system services from Windows NT.
1. Srvany.exe allows any Windows NT application to run as a service.

最后比较遗憾的是，尽管在windows服务里面也查看到了我已经注册到的服务，但是怎么都跑不起来，重启也是一个情况。顺带说一句，它这个方案真是麻烦又难用。于是继续搜索，后来终于找到了一个工具可以解决，一个叫**nssm**的工具（“the Non-Sucking Service Manager”），它会监控你安装的node服务，如果node挂了，nssm会自动重启它。可以看[官网](http://nssm.cc/ "nssm")深入了解。<!-- more -->

1. 进入官网下载对应系统版本，安装完成后，进入程序安装目录，nssw install +你的服务名（influx）
2. 接着它会弹出注册服务界面，里面需要配置相关的目录，按要求配置即可。**Path**中选择你的node.exe的安装路径，**Startup directory** 选择你的node应用的目录，即项目文件启动目录，**Argument**输入你的启动文件，我们项目是app.js

就绪过后，点击Install Service，它会弹出提示框提示服务已安装成功。

通过上面的步骤我们已经把需要的node服务已经注册成功，接着是使用。依然是进入nssm的程序安装目录，运行启动服务：
>nssm start influx

我们可以通过自己的客户端来连接服务器，测试是否安装服务成功。当然上面提到的start参数，对应肯定还有stop，所以欲想了解更多参数，可以去官网查看，里面很详细。