### Source Map

#### 作用

Javascript 变得越来越复杂，在上线之前代码都经过一定的转换，常见的转换有以下：

- 1、压缩，减少体积
- 2、变量名简写，减少字符
- 3、文件合并，减少 HTTP 请求数
- 4、代码转换比如 ES6+转换成 ES5

转换后的代码可能只有一行或几行，在执行时如果有报错，很难去 debug，而 source map 要解决的就是这个问题。

#### 什么是 Source map

简单来说，source map 就是一个信息文件，里面存着位置信息，也就是说，转换后的代码的每一个位置，所对应转换前的位置，有了它，出错的时候，出错工具(比如 Chrome 自带)将直接显示原始代码，而不是转换后的代码，给开发者带来很大方便。

目前只有 Chrome 浏览器支持这个功能，在 Developer Tools 的 setting 设置中，确认选中 Enable source map 开启该功能。

#### 如何启用

在转换后的代码尾部，加上一行：

```Javascript
    //@ sourceMappingURL=sourcemap所在的路径 改路径即可是网络上的也可以放在本地文件系统
```