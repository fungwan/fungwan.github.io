title: rabbitMQ安装笔记
date: 2015-8-8 17:44:12
tags:
- rabbitMQ 
categories: 服务器端开发
---

根据自己的实验结果，同网上的其他教程资料参考，自己也笔记一份搭建手册供以后生产环境使用！

# 一、Erlang安装 #

RabbitMQ是基于Erlang的，所以首先必须配置Erlang环境。

1. 安装依赖包，我的实验环境是需要安装**ncurses-devel**，至于其他的可按实际需要安装，这在后面erlang安装完成后有提示的

    yum install ncurses-devel
 
2. 2.下载Erlang源码otp_src_R15B03-1.tar.gz，并解压到指定位置

    tar -xzvf otp_src_R15B03-1.tar.gz

3. 进入安装目录，依次执行以下操作

    ./configure --prefix=/usr/local/erlang --without-javac //不用java编译，故去掉java避免错误
    
    make && make install

4. 上述编译安装无误后，配置环境变量

    vi /etc/profile

    export PATH=$PATH:/usr/local/erlang/bin


5. 在终端输入erl看可否进入交互界面，如下

    Erlang/OTP 18 [erts-7.0] [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]

<!-- more -->

# 二、RabbitMQ安装 #

关于rabbitMQ的安装也可以参照[官网](http://www.rabbitmq.com/build-server.html "rabbitMQ") 的教程，按照提示我们先要安装所需依赖等。


2.1 **安装Python**

1. 下载最近版本的python，替换系统默认2.6

    tar -xzvf Python-2.7.6.tgz

2. 进入安装目录，依次执行以下操作（即配置安装目录）
    
    ./configure --prefix=/usr/local/python27

3.  编译&&安装

    make && make install

4. 废弃旧pytho

    mv /usr/bin/python /usr/bin/python2.6.6.old 

5. 建立新版本python链接

    ln -s /usr/local/python27/bin/python /usr/bin/python


2.2**安装simplejson**

1. 下载相应版本压缩文件解压缩

    tar xvzf simplejson-2.2.1.tar.gz

2. 进入相应目录安装

    python setup.py install

2.3**安装xmlto**

1. yum方式安装

    yum install xmlto


2.4**安装rabbitMQ**

1. 下载相应版本压缩文件解压缩

    tar xvzf rabbitmq-server-3.1.5.tar.gz

2. 进入相应目录编译、安装

    make

	make install TARGET_DIR=/opt/rabbitmq SBIN_DIR=/opt/rabbitmq/sbin MAN_DIR=/opt/rabbitmq/man

3. 安装rabbitMQ相应插件（前提是rabbitMQ安装成功，有/etc/rabbitmq这个目录），以便后面用web后台监控服务器状态

    mkdir /etc/rabbitmq
    
    rabbitmq-plugins enable rabbitmq_management

4. 进入web后台

    rabbitmq-server start
    
    guest/guest 登录localhost:15672

**注意**上面的诸如rabbitmq-server的bash命令需要在$SBIN_DIR路径下执行。当然你也可以通过配置环境变量后执行。