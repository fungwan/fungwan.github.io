title: 解决Node在centos上无法访问
date: 2014-8-9 13:44:12
tags:
- node.js 
categories: 后端开发
---
将Node安装在了centos 上，发现在主机上并不能访问虚拟机上centos的node服务器，但是虚拟机能访问自己的web。各自都可ping通对方的IP。虚拟机采用的NAT的方式。后来查阅了资料，可能是防火墙的原因。
服务器的5858端口被防火墙堵了，可在主机通过命令：telnet server_ip 5858 来测试。

解决方案：

```javascript
/sbin/iptables -I INPUT -p tcp --dport 5858 -j ACCEPT
```

然后保存：
```javascript
/etc/rc.d/init.d/iptables save
```

重启防火墙

```javascript
/etc/init.d/iptables restart
```

再在主机访问OK！
