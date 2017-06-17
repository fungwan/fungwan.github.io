title: Node.js在Centos上的安装
date: 2014-8-8 17:44:12
tags:
- node.js 
categories: 后端开发
---
之前在windows上写Node,后面考虑到项目部署，于是需要移植到centos上。

环境需求：
1. centos版本为6.5 64位
2. Node安装0.10.24

步骤如下：

1.安装gcc编译

```javascript
yum -y install gcc gcc-c++ openssl-devel 
```

2.下载node.js
```javascript
a.wget http://nodejs.org/dist/v0.10.24/node-v0.10.24.tar.gz  
b.tar zxvf node-v0.10.24.tar.gz 
c.cd node-v0.10.24
```

3.配置编译安装

```javascript
./configure --prefix=/usr/local/node 
make && make install
```

4.配置Node环境
vim /etc/profile 

```javascript

#set nodejs env  
export NODE_HOME=/usr/local/node  
export PATH=$NODE_HOME/bin:$PATH  
export NODE_PATH=$NODE_HOME/lib/node_modules:$PATH  
```
 
qw!

```javascript
source /etc/profile 
```

5.重启
