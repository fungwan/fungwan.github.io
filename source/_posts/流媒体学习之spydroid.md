title: 解决spydroid-ipcamera视频串流的问题
date: 2015-2-14 15:44:12
tags:
- rtsp
categories: 后端开发
---
根据项目需求，要实现android端推送视频流，然后在客户端（这里是浏览器）里播放。后面选取的方案是采用Google Code上的一个开源项目：spydroid-ipcamera，它能在Android手机中建立一个精简的HTTP Server和RTSP Server。这跟以往的如需要类似red5流媒体服务器中转的解决方案不一样，它是在推送端直接建立服务器，让需要显示的客户端（浏览器）去连接它。<!-- more -->

在实作的过程当中，按照demo例子可以其正常运行，但是这里有一个小插曲，就是当初用vlc PC客户端去连接访问http地址或者rtsp地址时可正常访问，但是用网页的方式就是不行。后来通过单步调试才明白是浏览器上vlc插件似乎安装有问题，js代码里面提示vlc这个object找不到，继而addItem流失败。后面更新了下firefox和chrome的插件，然后开启一直启用方可正常！

似乎到这里就都ok了，但好事多磨，当我在浏览器嵌入2个vlc播放对象，分别连接两个不同的android设备时候，竟然发现在浏览器上vlc播放窗口出现‘串影’，即一个vlc窗口里面会间断性播放两个android设备的直播流，而另一个vlc窗口则毫无响应。当时大伙儿都以为可能是web端在处理vlc对象的时候出问题了，是代码错误，可是后面看了vlc文档似乎找不出什么原因。于是，又换了另一种测试方案，分别开两个浏览器，再各自浏览器只嵌入1个vlc对象，再来连接android设备，最后的结果出人意料，还是出现上述的bug。

这下使得我们将问题的出处指向了android客户端，没办法下载了它的源码，啃一啃再说。这里有一篇关于[spydroid](http://blog.csdn.net/xiaoliouc/article/details/8493161 ".")源码剖析的文章，会有一点帮助，此外还需要读一读关于rtsp协议的交互过程，才能更好理解代码的作用。参考这里，[关于RTSP的理解和例子说明 ](http://blog.chinaunix.net/uid-790245-id-2037516.html ".")。

最后，通过各种折腾，终于让我发现了这个bug的罪魁祸首！在RTSP交互命令的过程里面有这么一节点（**DESCRIBE**）：

    C->S:DESCRIBE request //客户端要求得到S提供的媒体初始化描述信息 
    S->C:DESCRIBE response //S回应媒体初始化描述信息，主要是sdp 

在上面code部分的第二行，server回复描述信息的内容里面，如下：

    RTSP/1.0 200 OK 
    Server: UServer 0.9.7_rc1 
    Cseq: 2 
    x-prev-url: rtsp://192.168.20.136:5000 
    x-next-url: rtsp://192.168.20.136:5000 
    x-Accept-Retransmit: our-retransmit 
    x-Accept-Dynamic-Rate: 1 
    Cache-Control: must-revalidate 
    Last-Modified: Fri, 10 Nov 2006 12:34:38 GMT 
    Date: Fri, 10 Nov 2006 12:34:38 GMT 
    Expires: Fri, 10 Nov 2006 12:34:38 GMT 
    Content-Base: rtsp://192.168.20.136:5000/xxx666/ 
    Content-Length: 344 
    Content-Type: application/sdp 
    
    v=0 //以下都是sdp信息 
    o=OnewaveUServerNG 1451516402 1025358037 IN IP4 192.168.20.136 
    s=/xxx666 
    u=http:/// 
    e=admin@ 
    c=IN IP4 0.0.0.0 
    t=0 0 
    a=isma-compliance:1,1.0,1 
    
    a=range:npt=0- 
    m=video 5006 RTP/AVP 96 //m表示媒体描述，下面是对会话中视频通道的媒体描述 
    a=rtpmap:96 MP4V-ES/90000 
    a=fmtp:96 
    profile-level-id=245;config=000001B0F5000001B509000001000000012000C888B0E0E0FA62D089028307 
    
    a=control:trackID=0//trackID＝0表示视频流用的是通道0

其中在m行中,中间的数字为服务端推荐客户端接收的端口,如果服务端不想这样做,可以将port置为0。对！问题就出现在这推荐端口上。demo的源码在实现它时，默认写死了5006，所以试想一下，在同一台pc机上，IP地址一样，两个vlc使用的又都是默认推荐端口5006，这样android端在往客户端推流的时候势必会出现串流的问题。解决的办法就是，如果想在一台pc机上使用多个vlc播放spydroid直播流的话，那么android在推荐给客户端的端口上就不能写死，即确保唯一性就行！





