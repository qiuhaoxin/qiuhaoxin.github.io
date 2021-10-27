### package.json 文件详解 <!-- {docsify-ignore} -->

package.json 大家都不陌生，在现在几乎每个前端的项目里随处可见，它是项目描述文件，顾名思义是用来描述一个项目的。

#### 创建

- npm init -y
  后面的参数 -y 可选，意思是默认生成选项

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

dependencies: 指定项目运行时的依赖模块
devDependencies：指定项目开发阶段所依赖的模块

上面两个模块都是一个对象值，由模块名和版本号构成，对应的版本号可以加上限定符合，限定符具体有以下几种：

```Javascript
    1、指定版本比如1.2.2，遵循"大版本.次要版本.小版本"的格式，安装的时候只安装指定版本
    2、波浪号 + 指定版本 如：~1.2.2 ，表示安装1.2.x的最新版本，不能低于1.2.2。但不能安装1.3.x,也就是说安装的版本不能改变大版本号和次要版本号。
    3、插入号 + 指定版本 如^1.2.2,表示安装1.x.x版本（不低于1.2.2）,但不安装2.x.x。也就是说安装是不改变大版本号，需要注意的是，如果大版本号为0，则插入号跟波浪号行为相同，因为在开发初始阶段，即使是次要版本不同也会带来不兼容。
    4、latest，安装最新版本
```

peerDependencies：就是用来供插件指定其所需要的主工具的版本
。有时候你的项目和所依赖的模块，会共同依赖另一个模块，但所依赖的版本可能会不一样。比如说你的项目依赖模块 A 和模块 B 的 1.0 版本，模块 A 本身又依赖模块 B 的 2.0 版本。再大多数的情况下，这都不构成问题，但如果将这种依赖关系暴露给用户的时候，就会产生问题。

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

config:用于添加命令行的环境变量

```package.json
    {
        "name":"foo",
        "config":{"port":"8080"},
        "scripts":{
            "start":"node server.js"
        }
    }
```

假如要启动一个服务器，使用到 port：

```Server.js
    const http=require('http');
    http.createServer(...)
    .listen(process.env.npm_package_config_port);
```

当用户执行命令行脚本时，就可以得到值，并可以改变值

```cli
    npm run start

    //可以配置
    npm config set foo.port=8081
```

type:用 webpack 构建时，有时会指定 type 为 module 或 commonjs，webpack 会启动语法一致性检查，当为 module 时，模块化语法不允许用 CommonJS(不能用 exports、module.exports 或 require)，后缀命名不能是.cjs 而是.mjs 或.js。当为 commonjs 时不能用 ESM 语法（不能用 import、export）另外 webpack2.0 时已经原生支持 import 和 export 了（不用引用 babel 也能处理），但 ES6+的语法还是需要 babel 来处理
