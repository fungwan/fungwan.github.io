title: package.json里面的版本管理
date: 2014-10-25 19:44:12
tags:
- package.json
categories: 后端开发
toc: false
---
在学习和使用node的过程中，常常会接触到package.json里面的配置，不管是自己填写还是去阅读人家的文件，刚开始都是一头雾水，此篇笔记只是针对里面的**dependencies**属性。因为我们在更新或者使用人家第三方模块的时候（npm install），最关键的就是dependencies里面的版本值。举一个小例子：

    "dependencies": {
    "websvr": "0.1.x",
    "websvr-redis": "0.x.x",
    "mustache": "~0.7.2"，
    "connect-multiparty": "^1.1.0" }

这里面头两个模块的依赖都好理解，即websvr匹配0.1.0,0.1.1等版本，不会匹配如0.2.0之类的。关键后面的两个模块依赖，有两个特殊符号~和^,它们的区别分别如下：

- ~会匹配最新的子版本（中间那个数字），比如~0.7.2会匹配所有的0.7.x版本，但不匹配到0.8.0及以上;
- ^会匹配最新的主版本（第一个数字），比如^1.1.0将会匹配所有的1.x.x版本，2.0.0就略过。

最后，附上版本控制的其他范围：
    
    version 必须严格匹配到 version 版本
    >version 必须大于 version 的版本
    >=version 大于等于 version 的版本
    <version 小于
    <=version 小于等于
    ~version “几乎相同的版本（Approximately equivalent to version）” 参见semver(7)
    ^version “兼容的版本” 参见 semver(7)
    1.2.x 匹配1.2.0, 1.2.1, 之类., 但不匹配 1.3.0
    http://... 以URL地址作为依赖
    * 匹配任意版本
    "" (就是一个空白字符empty string) 同 *，任意版本
    version1 - version2 同 >=version1 <=version2.
    range1 || range2  range1 或者 range2 的任一版本.
    git... 以GIT地址作为依赖
    user/repo 以GitHub地址作为依赖
    tag 匹配一个以 tag 标记并发布的版本，参见 npm-tag(1)
    path/path/path 以本地地址作为依赖
    