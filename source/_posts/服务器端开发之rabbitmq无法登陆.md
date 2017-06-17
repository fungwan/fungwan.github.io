title: 解决rabbitMQ无法远程登录
date: 2015-8-25 11:50:38
tags:
- rabbitMQ 
categories: 后端开发
---

搭建好了rabbitmq集群后，自己想测试一下，于是通过rabbitmq的node.js lib去连了一下，我这里直接连的是proxy节点，但是迟迟没反应，没有收到回复消息，于是跑上去看日志，上面确实是收到了我的连接请求，但是proxy节点却在转发的时候出了问题，它始终连接不上下面的节点，难道是防火墙的问题，不会吧？我三台机子可是全部关闭了的，后来几经查证，确认了是rabbitmq自身机制问题引发的。

原因是出于安全的考虑，guest这个默认的用户只能通过http://localhost:15672 来登录，不能使用IP地址登录，也就是不能远程访问，这样的话，对于服务器上没有安装桌面的情况是不方便管理维护的（不用代码的方式）。<!-- more -->

要解决这个问题需要配置远程登录权限，我们可以先以guest的身份登录web后台，然后配置相应权限即可。步骤如下：

1. 进入“admin”标签页，然后点击“Add a user ”
2. 输入对用的帐号密码，然后选择用户角色（一定要选择）
3. 为了授权该用户对VirtualHost"/" 的访问，用户添加之后，需要对该用户进行授权，不然运行会出现错误
4. 在Admin标签页下点击新增的用户"admin"，进入授权页面，默认直接点击"set permission"即可

用户以及授权添加完成之后，在rabbitmq.config.example文件中，添加以下内容，保存后重启RabbitMQ服务：

    ……
    [
     {rabbit,
      [%%
      %% Network Connectivity
      %% ====================
      %%
      %% By default, RabbitMQ will listen on all interfaces, using
      %% the standard (reserved) AMQP port.
      %%
      {tcp_listeners, [5672]},
      {loopback_users, ["admin"]},
    ……
      ]}
    ].


最后在浏览器中输入http://remoteIP:15672实现通过IP地址访问，成功登录。