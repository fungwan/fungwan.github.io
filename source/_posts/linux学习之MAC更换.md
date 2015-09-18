title: 解决clone过后的虚拟机网卡上网相关问题
date: 2015-1-6 12:44:12
tags:
- 日常问题 
categories: linux
---

当我把现有的centos虚拟机文件复制或clone一份后，然后再运行复制后的.vmdk文件，会发现系统加载的时候会提示相关错误信息，大致就是网卡不能启动，

    Bringing up interface eth0: Device eth0 has different MAC address than expected,ignoring.

究其原因就在于虚拟机镜像拷贝的问题，因为复制.vmdk文件是将虚拟机完全copy了一份（包括MAC地址），由于新的硬件平台网卡MAC与系统中网卡MAC不一致，因此就会有上述的结果显示。这时候你在系统里面ifconfig是得不到当前主机IP地址的信息。<!-- more -->

从网上看了很多解决办法，试了很多次，最后选定了其中一个，而且屡试不爽，于是贴出来供以后参考:

1. ifconfig eth0 down
2. cd /etc/sysconfig/network-scripts
3. vim ifcfg-eth0 //修改其中的"HWADDR=xx:xx:xx:xx:xx:xx"为"MACADDR=xx:xx:xx:xx:xx:xx"
4. ifconfig eth0 up
5. service network start

**注意:**关键词HWADDR和MACADDR是有区别的,不然可能会修改失败！新的mac地址可以在虚拟机的配置里面查看，然后将其替换。

另外，还看到其他地方对于70-persistent-net.rules这个文件也作了修改，它里面是IP地址与网卡的匹配文件，路径在/etc/udev/rules.d/，基于了上面的方案，我是直接将其删除没有作修改，反正它会在开机时根据网络情况自动生成。

还有一点我遇到的问题是（并不代表大家会遇到哦），在mac地址更新成功后，ifconfig也能显示正确结果，但是最后居然不能上网，当时就纳闷了，以为是防火墙问题，于是各种ping，发现与真实主机都可平通，甚至于我把防火墙也关了，然后ping www.baidu.com不行，但是ping百度的ip地址可以，难道是dns的配置问题，不会啊？我都是自动获取的，压根就没改过。后来经过数次排查，终于找到原因了，原来在我主机的服务里面不知什么时候把虚拟机的各种服务给关掉了（包括上网的NAT），于是全部启动，再试，成功了！额，顿时感觉好囧！

