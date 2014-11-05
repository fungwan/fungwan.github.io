title: npm install经常失败解决方法
date: 2014-11-05 17:44:12
tags:
- node.js
- npm
categories: node.js
---

之前在使用node.js的好基友npm包管理器install模块的时候，间断性的失败，而且是屡试不爽，后来网上了解到和我有相同遭遇的朋友们不在少数。原因就在于npm包管理服务器在米国，而我们有骄傲的GFW，故导致上述install不稳定的情况。所以，为了更好的使用体验采用了淘宝的 NPM 镜像，它是一个完整的npmjs.org镜像。


*1.通过定制的 cnpm 命令行工具代替默认的 npm
```javascript
npm install -g cnpm --registry=http://registry.npm.taobao.org
```
*2.安装具体模块

```javascript
 cnpm install [name]
```
`tips`:当安装的时候发现安装的模块还没有同步过来, 淘宝 NPM 会自动在后台进行同步, 并且会让你从官方 NPM registry.npmjs.org 进行安装. 下次你再安装这个模块的时候, 就会直接从 淘宝 NPM 安装了.

*3.同步模块（直接通过 sync 命令马上同步一个模块）
```javascript
cnpm sync connect
```