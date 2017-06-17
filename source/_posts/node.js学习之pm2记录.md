title: PM2的学习记录
date: 2016-2-18 13:22:12
tags:
- node.js
categories: 后端开发
toc: false
---

最近项目上线，准备上pm2，一个带有负载均衡功能的Node应用的进程管理器。抛弃forever了，哈哈。

主要特性：

 1. 内建负载均衡（使用Node cluster 集群模块）
 2. 后台运行
 3. 0秒停机重载，我理解大概意思是维护升级的时候不需要停机.
 4. 具有Ubuntu和CentOS 的启动脚本
 5. 停止不稳定的进程（避免无限循环
 6. 控制台检测
 7. 提供 HTTP API
 8. 远程控制和实时的接口API ( Nodejs 模块,允许和PM2进程管理器交互 )


下面是有用的命令：

> npm install pm2 -g     # 命令行安装 pm2 
pm2 start app.js -i 4 #后台运行pm2，启动4个app.js 
pm2 start app.js --name my-api # 命名进程
**pm2 list**               # 显示所有进程状态
**pm2 show 0**               # 显示某个id的进程状态
**pm2 monit**              # 监视所有进程
pm2 logs               #  显示所有进程日志
pm2 stop all           # 停止所有进程
pm2 restart all        # 重启所有进程
pm2 reload all         # 0秒停机重载进程 (用于 NETWORKED 进程)
pm2 stop 0             # 停止指定的进程
pm2 restart 0          # 重启指定的进程
pm2 startup            # 产生 init 脚本 保持进程活着
pm2 web                # 运行健壮的 computer API endpoint
pm2 delete 0           # 杀死指定的进程
>pm2 delete all         # 杀死全部进程

运行进程的不同方式：
<!-- more -->
>pm2 start app.js -i max  # 根据有效CPU数目启动最大进程数目
>pm2 start app.js -i 3      # 启动3个进程
>pm2 start app.js -x        #用fork模式启动 app.js 而不是使用 cluster
>pm2 start app.js -x -- -a 23   # 用fork模式启动 app.js 并且传递参数 (-a 23)
>pm2 start app.js --name serverone  # 启动一个进程并把它命名为 serverone
>pm2 stop serverone       # 停止 serverone 进程
>**pm2 start app.json**        # 启动进程, 在 app.json里设置选项
>pm2 start app.js -i max -- -a 23  #在--之后给 app.js 传递参数
>pm2 start app.js -i max -e err.log -o out.log  # 启动生成一个配置文件

再补充一个app.json的写法，cluster与fork混用，启用多个实例：
```python
[{
  "name"        : "server1",
  "script"      : "bin/www",
  "cwd"         : "./server1",
  "error_file" : "server1-err.log",
  "out_file"   : "server1-out.log",
  "pid_file"   : "server1.pid",
  "instances"  : 0,
  "exec_mode"  : "cluster",
  "env": {
      "NODE_ENV": "production"
  }
},
{
  "name"        : "server2",
  "script"      : "bin/www",
  "cwd"         : "./server2",
  "node_args" : "--harmony"
  "error_file" : "server2-err.log",
  "out_file"   : "server2-out.log",
  "pid_file"   : "server2.pid",
  "instances"  : 1,
  "exec_mode"  : "fork",
  "env": {
      "NODE_ENV": "production"
  }
}]
```
