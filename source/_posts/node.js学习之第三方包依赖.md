title: 理解package.json中devDependencies和dependencies的区别
date: 2015-7-28 18:22:12
tags:
- node.js
categories: 后端开发
toc: false
---

用node写项目经常会用到第三方的包，阅读人家工程代码时留意到package.json里面除了我们常见的dependencies节点属性外，还看到了另外一个devDependencies，那么它们二者有什么区别呢？

dependencies 程序正常运行需要的包，是需要发布到生产环境的。devDependencies 是开发需要的包，比如一些单元测试的包之类的。其中前者依赖的项该是正常运行该包时所需要的依赖项，而后者则是开发的时候需要的依赖项，像一些进行单元测试之类的包（jslint）。

换言之，在使用 --save-dev 安装的插件它是被写入到 devDependencies对象里面去，而使用 --save 安装的插件，则被写入到 dependencies 对象里面去。

如果将包下载下来在包的根目录里运行npminstall默认会安装两种依赖，如果只是单纯的使用这个包而不需要进行一些改动测试之类的，可以使用npm install --production，只安装dependencies而不安装devDependencies。反之npm install xxx -dev