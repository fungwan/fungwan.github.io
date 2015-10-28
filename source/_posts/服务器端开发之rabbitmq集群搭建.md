title: rabbitMQ集群搭建
date: 2015-8-18 16:48:15
tags:
- rabbitMQ 
categories: 服务器端开发
---

本文笔记的是rabbitMQ集群搭建（镜像模式），同参考教程一样，设计模型为，在一个集群里，有3台机器，其中1台使用磁盘模式，另1台使用内存模式。内存模式的节点，无疑速度更快，因此客户端（consumer、producer）连接访问它。而磁盘模式的节点（RabbitMQ启动后，默认是磁盘节点），由于磁盘IO相对较慢，因此仅作数据备份使用，另外一台作为反向代理。接下来是具体配置：


1. 修改主机名

    输入vim /etc/sysconfig/network后，将HOSTNAME后面的值改为想要设置的主机名

	输入vim /etc/hosts后，将localhost.localdomain改为想要设置的主机名。

	reboot命令，重新启动服务器后，输入hostname查看主机名是否修改

	修改主机名的意义在于以后以此区分各个节点

 
2. 修改host文件

    在安装好的两台节点服务器中，分别修改/etc/hosts文件，如：

    	172.16.3.32 queue
    
    	172.16.3.107 influx

    	172.16.3.109 proxy


3. 设置每个节点Cookie

	Rabbitmq的集群是依赖于erlang的集群来工作的，所以必须先构建起erlang的集群环境。Erlang的集群中各节点是通过一个magic cookie来实现的，这个cookie存放在 /var/lib/rabbitmq/.erlang.cookie 中，文件是400的权限。所以必须保证各节点cookie保持一致，否则节点之间就无法通信。<!-- more -->

4. 使用detached参数独立运行,查看各节点信息
			
			queue# rabbitmq-server -detached
			queue# rabbitmqctl cluster_status

            Cluster status of node rabbit@queue ...
			[{nodes,[{disc,[rabbit@queue]}]},
			{running_nodes,[rabbit@queue]},
			{partitions,[]}]
			...done.


5. 将influx作为内存节点与queue连接起来

	 在influx上，执行如下命令：rabbitmqctl join_cluster --ram rabbit@queue   
	
6. 接着是配置镜像模式

	上面配置RabbitMQ默认集群模式，但并不保证队列的高可用性，尽管交换机、绑定这些可以复制到集群里的任何一个节点，但是队列内容不会复制，虽然该模式解决一部分节点压力，但队列节点宕机直接导致该队列无法使用，只能等待重启，所以要想在队列节点宕机或故障也能正常使用，就要复制队列内容到集群里的每个节点，需要创建镜像队列
	
	在任意一个节点上执行：
	rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'

7. 增加负载均衡器

	关于负载均衡器，商业的比如F5的BIG-IP，Radware的AppDirector，是硬件架构的产品，可以实现很高的处理能力。但这些产品昂贵的价格会让人止步，所以我们还有软件负载均衡方案。互联网公司常用的软件LB一般有LVS、HAProxy、Nginx等。LVS是一个内核层的产品，主要在第四层负责数据包转发，使用较复杂。HAProxy和Nginx是应用层的产品，但Nginx主要用于处理HTTP，所以这里选择HAProxy作为RabbitMQ前端的LB。

	HAProxy的安装使用非常简单，在Centos下直接yum install haproxy，然后更改/etc/haproxy/haproxy.cfg 文件即可，文件内容大概如下：



>     defaults
>     modehttp
>     log global
>     option  httplog
>     option  dontlognull
>     option http-server-close
>     option forwardfor   except 127.0.0.0/8
>     option  redispatch
>     retries 3
>     timeout http-request10s
>     timeout queue   1m
>     timeout connect 10s
>     timeout client  1m
>     timeout server  1m
>     timeout http-keep-alive 10s
>     timeout check   10s
>     maxconn 3000
>      
>     listen rabbitmq_cluster 0.0.0.0:5672
>     mode tcp
>     balance roundrobin
>     server   rqslave1 172.16.3.32:5672 check inter 2000 rise 2 fall 3   
>     server   rqslave2 172.16.3.107:5672 check inter 2000 rise 2 fall 3 
	

参考文章[在这](http://www.cnblogs.com/flat_peach/archive/2013/04/07/3004008.html)。