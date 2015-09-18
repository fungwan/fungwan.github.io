title: node.js获取本机IP地址
date: 2015-1-7 10:29:15
tags:
- node.js
categories: node.js
toc: false
---

在node.js里面获取本机IP很简单，在OS模块里面，直接调用networkInterfaces()，就可以列出你机子的所有网络接口的信息,它返回的是一个数组，如官网列出的信息：

```javascript
{ lo0: 
   [ { address: '::1', family: 'IPv6', internal: true },
     { address: 'fe80::1', family: 'IPv6', internal: true },
     { address: '127.0.0.1', family: 'IPv4', internal: true } ],
  en1: 
   [ { address: 'fe80::cabc:c8ff:feef:f996', family: 'IPv6',
       internal: false },
     { address: '10.0.1.123', family: 'IPv4', internal: false } ],
  vmnet1: [ { address: '10.99.99.254', family: 'IPv4', internal: false } ],
  vmnet8: [ { address: '10.88.88.1', family: 'IPv4', internal: false } ],
  ppp0: [ { address: '10.2.0.231', family: 'IPv4', internal: false } ] }

```

据此，我们可以根据实际需要取到相应的值：

```javascript

function getIPAdress(){

    var interfaces = require('os').networkInterfaces();

    for(var devName in interfaces){

        var iface = interfaces[devName];
        for(var i=0;i<iface.length;i++){
            var alias = iface[i];
            if(alias.family === 'IPv4' && alias.address !== '127.0.0.1' && !alias.internal){
                return alias.address;
            }
        }
    }
}

```

<!-- more -->
还有因为javaweb项目中的其他地方需要此功能，所以这里顺便说一下关于java版本的获取方式：

```javascript

Enumeration allNetInterfaces = NetworkInterface.getNetworkInterfaces();
		InetAddress ip = null;
		
		String strIp = "";
		
		while (allNetInterfaces.hasMoreElements())
		{
			NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
			if (netInterface.getName().equals("lo")) {
				System.out.println("回环地址：" + netInterface.getName());
                continue;
            }else{
    			Enumeration addresses = netInterface.getInetAddresses();
    			while (addresses.hasMoreElements())
    			{
    				
    				ip = (InetAddress) addresses.nextElement();
    				if (ip != null && ip  instanceof Inet4Address) 
    				{
    					if(netInterface.isVirtual()){
    						continue;
    					}
    					System.out.println("本机网卡描述：" + netInterface.getDisplayName());
    					
    					if(netInterface.getDisplayName().indexOf("Virtual") == -1){
    						System.out.println("本机的IP： " + ip.getHostAddress());
    						strIp = ip.getHostAddress();
    					}
    					
    				} 
    			}
            }
			
		}

```

通过发现，好像它们都没有直接的办法可以区分出虚拟网卡的IP地址。