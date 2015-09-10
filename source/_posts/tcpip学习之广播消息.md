title: 应用于广播的解决方案
date: 2015-6-20 17:32:11
tags:
- Tcp/Ip
categories: TCP/IP
---

以前android终端连接服务器的地址都是写在了代码里（不可取）或者配置文件中，当然在平日开发中为了方便，问题当然不大，主要是我们的局域网中服务器的IP地址基本不会改变，要么都是手动设置了IP。但是如果一旦作为产品卖给客户，问题就凸显出来了，因为我们到时候是不清楚客户方的网络情况，不可能到一个地方都去改代码然后打包，这显然是不现实的。<!-- more -->

为此大家开会商讨后，提出了两个方案，第一个是在android终端启动界面处添加一个可以设置服务器连接地址的按钮，让用户可以设置服务器的连接地址。虽然这可以解决修改代码、重新打包的琐事，但是又是一个新的问题出来了，如果涉及的终端有很多怎么办，也是需要人力一个个去设置吗？所以这只能是一个备选方案作参考。

那第二个方案就是采用广播消息的形式，有点类似于局域网arp协议广播一样。意思就是，前提当服务器与各个终端都在一个网段，每次终端登陆连接的时候，终端就向这个网段里面发送一个广播消息，内容大概就是“Who is server?”，当然我们讨论的这个解决方案是确保处于一个安全的局域网里面（内部网），至于类似于arp欺骗忽略之。这时如果是服务器的话，收到了这个消息就返回响应消息给询问的终端，内容类似“Server is here！”，这样发送广播消息的客户端就会收到应答消息，那么对应的就知道了服务器的IP地址，然后就水到渠成的连接并与服务器通信了。写个node.js实现吧，当然这是用udp作广播消息：

**服务器端**

```javascript
var dgram = require("dgram");
var udpServer = dgram.createSocket("udp4");

udpServer.on("error", function (err) {
    logger.error("server error:\n" + err.stack);
    udpServer.close();
});

udpServer.on("message", function (msg, rinfo) {
    logger.trace("server got: " + msg + " from " +
        rinfo.address + ":" + rinfo.port);

    if(msg === 'whoIsServer'){

        var message = new Buffer('serverIsHere');

        udpServer.send(message, 0, message.length, 41234, rinfo.address, function(err, bytes) {
            //socket.close();
        });
    }
});

udpServer.on("listening", function () {
    var address = udpServer.address();
    logger.trace("Magnum udp server listening " +
        address.address + ":" + address.port);
});

udpServer.bind(41234);
```

**客户端**

```javascript
var dgram = require("dgram");
var udpClient = dgram.createSocket("udp4");

var message = new Buffer('Who is server');

udpClient.send(message, 0, message.length, 41234, rinfo.address, function(err, bytes) {
          udpClient.close();
});
```

以上就是针对项目中上线可能会遇到的问题，当然上面提到的第一个方案也可作为保留，以备万一嘛。