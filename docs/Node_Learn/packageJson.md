### package.json 文件详解 <!-- {docsify-ignore} -->

package.json 大家都不陌生，在现在几乎每个前端的项目里随处可见，它是项目描述文件，顾名思义是用来描述一个项目的。

#### 创建

- npm init -y
  后面的参数可选，意思是默认生成选项

#### 字段详解

name: 项目名称
version：项目版本号
description：项目描述
keywords：字符串数组，有助于人们发现你的包
homepage：项目主页
bugs：项目问题跟踪器的 url 和/或应报告问题的电子邮件地址
license：许可
author：作者
contributors：贡献者
repository：代码存储位置

```Javascript
    repository:{
        "type":"git",
        "url":"https://github.com/npm/cli.git"
    }
```

main：主要入口，是 package.json 的另一种元数据功能，它可以用来指定加载的入口文件。假如你的项目是一个 npm 包，当用户安装你的包，并调用 require('your-module')时，返回的是 main 字段指定文件的 module.exports 属性，如果不知道该字段，默认的是项目中根目录的 index.js 文件
browser：如果你的模块是在浏览器环境下用的，优先使用这个字段而不是 main 字段，这有助于提醒用户该包有可能依赖 Node 环境下不能用的一些关键字比如 window、document
bin：自定义命令，很多时候，一个包都有一个或多个想安装到 PATH 的可执行文件，指定了该字段，在安装该包时，npm 会将该文件符号链接到 prefix/bin 全局安装或./node_modules/.bin/下局部安装。

```Javascript
    {
        name:'react-cli-library',
        version:'0.1.1',
        bin:{
            "react-cli":"./lib/index.js"
        }
    }
```

包名是:react-cli-library ,安装完成后，执行 react-cli 的命令，就会执行该包 lib 文件下的 index.js 文件。
注意：包名和命令名不一定要一致，这取决于 bin 字段的配置

engines:可以用来指定 node 或 npm 的版本，避免因为版本不兼容而产生问题

```Javascript
    "engines":{
        "node":"> 8.2.4"
    }
```

但这个字段只起到一个说明的作用，并不影响其他依赖的安装

os：模块适用系统，假如我们开发的模块只能适用某个系统下，需要包装不适应的系统不会安装该模块，从而避免错误。该属性可以指定模块适用的系统，或者指定不能安装的系统的黑名单（会报错）。

```Javascript
    "os" : [ "darwin", "linux" ] # 适用系统
    "os" : [ "!win32" ] # 黑名单
```

cpu：与上面的 os 差不多，用来指定模块安装的环境

```Javascript
    "cpu" : [ "x64", "ia32" ] # 适用 cpu
    "cpu" : [ "!arm", "!mips" ] # 黑名单
```

private：定义私有模块，如果是公司非开源的项目，一般会指定 private 字段值为 true，而 npm 是拒绝发布私有模块的。确保非开源项目不会发布到 npm 库上去
