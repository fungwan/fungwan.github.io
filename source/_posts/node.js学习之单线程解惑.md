title: 解惑node中的单线程
date: 2014-12-30 12:44:12
tags:
- node.js
categories: node.js
---
这里说一下node.js中的单线程缘由还是基于前段时间公司内部培训同事提出的问题：在网络通讯编程中，按照以往的语言或平台，应对成百上千的socket连接时，往往都会利用多线程的特性，对每个连接开启子线程分别接收数据再处理。那么在node.js中它是单线程，是基于事件响应的，即将每个的连接、处理事件会放到一个事件队列里面依次处理。那么问题来了，同事说如果某一个连接上来以后，一直在给服务器发送数据，那么排在事件队列后面的连接怎么处理，会一直等待吗？这个当然不是，下面我先用一个例子来说明一下这个问题的答案，方便理解。

我们准备模拟这样一个场景：一个客户端向服务器发起tcp连接，成功之后，客户端一直给服务器发送数据，而服务器收到数据后只是作一个回显。然后，我们再用一个客户端向该服务器发起tcp连接，同样发送数据，这时看服务器会不会因为接收上一个连接的持续发送数据，而不能去处理第二个tcp连接的数据发送。<!-- more -->

还是用熟悉的node.js撰写测试代码，首先是服务器server.js，如下：

```javascript

var net = require('net');
var timeout = 20000;//超时
var listenPort = 9911;//监听端口

var server = net.createServer(function(socket){
    console.log('connect: ' +
        socket.remoteAddress + ':' + socket.remotePort);

    //接收到数据
    socket.on('data',function(data){
		console.log(socket.remotePort + " :写入成功！")	    
		socket.write('哈哈，服务器我收到啦!' + data);
    });

    //数据错误事件
    socket.on('error',function(exception){
        console.log('socket error:' + exception);
        socket.end();
    });
    //客户端关闭事件
    socket.on('close',function(data){
        console.log('close: ' +
            socket.remoteAddress + ' ' + socket.remotePort);
    });

}).listen(listenPort);


```

然后是第一个客户端client1.js，我准备用一个大循环模拟持续发送数据的场景：

```javascript
var net  = require('net');

var port = 9911;
var host = '127.0.0.1';

var client= new net.Socket();
client.connect(port,host,function(){

    for(var x = 0; x < 1000000; ++x){
	    client.write('我是1号客户端');
	}
});

client.on('data',function(data){
    console.log('recv data:'+ data);
});

```

接着是另一个客户端client2.js，正常发送数据：

```javascript
var net  = require('net');

var port = 9911;
var host = '127.0.0.1';

var client= new net.Socket();

client.connect(port,host,function(){
	client.write('我是2号客户端！');
});

client.on('data',function(data){
    console.log('recv data:'+ data);
});

```

现在我们先运行client1.js,因为客户端一直在发送数据，所以发现客户端控制台一直在回显服务器传回来的数据，然后运行client2.js,不出意外，2的控制台也回显了服务器发回的数据！说明我们这样的测试是达到了实验要求。那这究竟是为什么呢？我们可以这样想，虽然你客户端1一直在往服务器发数据，但在服务器端它会触发很多data事件来接收处理，抽象一点来想，就好比这样的队列：data1——data1——data1——data1....等很多这样客户端1的data事件，那么如果此时有客户端2连上来并发送数据，队列就变成了：data1——data1——data1——data2——data1——data1——....，所以在队列里面只要依次处理完了data1的事件，总会轮到data2，即客户端2的数据处理。

那什么情况会出现同事说的那种情况呢？可能大家这会都应该知道了，如果在队列中，如果某一个data1的数据处理时间过长，或者遇到大循环，那么很久时间才会处理队列中的data2.比如，我们把服务端程序server.js改一下:

```javascript

var net = require('net');
var timeout = 20000;//超时
var listenPort = 9911;//监听端口

var server = net.createServer(function(socket){
    console.log('connect: ' +
        socket.remoteAddress + ':' + socket.remotePort);

    //接收到数据
    socket.on('data',function(data){
		for(var x = 0; x < 10000000 ; ++x){
	    	console.log('模拟计算，阻塞主线程');
		}   
		socket.write('哈哈，服务器我收到啦!并处理完成' + data);
    });

    //数据错误事件
    socket.on('error',function(exception){
        console.log('socket error:' + exception);
        socket.end();
    });
    //客户端关闭事件
    socket.on('close',function(data){
        console.log('close: ' +
            socket.remoteAddress + ' ' + socket.remotePort);
    });

}).listen(listenPort);


```
上面代码主要改动在，服务端接收数据过后，我在里面添加了一个大循环，模拟超时计算。同样按之前的方式运行程序，我们来看结果。这一次，不光是客户端2，连1的控制台都没有回显之前的字符提示了，很明显在处理data1的事件时耗时太久，已经完全阻塞了后面队列的事件处理。

从这次试验来看，我们也证实了node.js这种基于事件的单线程平台不适用于复杂密集的cpu运算的开发，不然迟早你的程序会卡死崩溃。