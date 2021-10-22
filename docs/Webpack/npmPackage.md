### 如何基于 Webpack 制作一个 NPM 包 <!-- {docsify-ignore} -->

webpack 不仅仅可以支持运行一个应用的构建，还可以构建 npm 包

#### 发布到 NPM 的包有以下几个特点：

- 1、每个模块的根目录下都必须有一个描述该模块的 package.json 文件，该文件描述了模块的入口文件是哪个，依赖哪些模块等
- 2、模块中以 Javascript 文件为主，但又不限于 Javascript 文件，如一些 UI 组件就可以包含图片、CSS 文件等
- 3、模块中大多采用模块化规范，因为你的模块可能依赖别的模块，而别的模块又可能依赖你这个模块，采取得比较通用的是 CommonJS 模块规范，上传到 NPM 的模块最好遵循该模块化规范

#### 实际开发中可能遇到的问题

- 1、源代码采用 ES6+语法来编辑，但发布到 NPM 上的语法要求是 ES5 的，并且遵循 CommonJS 规范。如果发布的代码是经过转换的，要提供 Source Map 以提供方便的调试
- 2、如果是 UI 组件，也要把相关的资源打包到发布的模块中，比如图片和 CSS 样式文件
- 3、尽量减少冗余代码，减小发布出去包的大小
- 4、发布出去的代码不能含有其依赖的模块的代码，而是让用户可选择性去安装，比如组件库里依赖了 React 库，但打包出来的包不应内嵌 React 库，这样是为了避免重复安装 React

#### 解决方案

针对上述的问题，逐个解决，并以编辑一个简单的 React 组件库举例：

假如发布到 NPM 仓库的 React 组件库代码目录结构如下：

源码：

```Javascript
    // src/index.js
    import React,{Component} from 'react';
    import './index.css';
    //导出该组件，以供其他模块使用
    export default class MyComponent extends Component{
        render(){
            return <div>Hello World!!!</div>
        }
    }
```

可以这样使用：

```Javascript
    // ES6 模块导入
    import MyComponent from 'ReactTest';
    import 'ReactTest/lib/index.css';
    //或者通过require
    const MyComponent =require('ReactTst')
    import React from 'react';
    import {render} from 'ReactDOM';

    render(<MyComponent />,document.getElementById('app'));
```

对于要求 1，我们可以这样做：

- 使用 babel-loader 对 ES6+的语法进行转换成 ES5，
- 通过开启 devtool:source-map,输出 Source Map 支持调试
- 设置 output.libraryTarget="commonjs2",使输出的代码符合 commonjs 的模块规范，以供其他模块导入使用
  相关的 Webpack 配置如下：

```webpack.config.js
    module.exports={
        output:{
            libraryTarget:'commonjs2',
        },
        devtool:'source-map',
        module:{
            rules:[
                {
                    test:/\.js$/,
                    use:[
                        'babel-loader',
                    ]
                }
            ]
        }
    }

```

对于要求 2，可以通过 css-loader 和 mini-css-extract-plugin 来提取相关 CSS 样式为单独文件：

```webpack.config.js
    const MiniCSSExtractPlugin=require('mini-css-extract-plugin');
    module.exports={
        ...
        module:{
            rules:[
                {
                    test:/\.css$/,
                    use:[
                        MiniCSSExtractPlugin.loader,
                        'css-loader',
                    ]
                }
            ]
        },
        plugins:[
            new MiniCSSExtractPlugin({
                filename:'[name].css',
            })
        ]
    }
```

对于要求 3，我们在使用 babel 来将 ES6+的语法转换成 ES5 的时候，会注入很多辅助函数，比如实现类继承：

```Javascript
    class MyComponent extends Component{

    }
```

在上面代码被转换成能运行的 ES5 代码，需要

```Javascript
    babel-runtime/helpers/createClass //用于实现 class 语法
    babel-runtime/helpers/inherits // 用于实现 extends 语法
```

默认情况下，Babel 会在每个实现类继承的模块里<strong>内嵌</strong>上面两个辅助函数的代码,那么这些辅助函数便会多次重复出现，造成冗余。为了不让这些函数重复出现，可以在依赖他们的时候通过 require('babel-runtime/helpers/createClass')的方式去导入，这样就可以让它们只出现一次。可以使用 Babel 的 babel-plugin-runtime-transform 来实现。

在 Babel.config.js 修改相关配置：

```Babel.config.js
    module.exports={
        "plugins":[
            "@babel/runtime-transform"
        ]
    }
```

注:由于 runtime-transform 依赖 babel-runtime，所以在安装@babel/runtime-transform 时也要同时安装 babel-runtime

对于要求 4，可以通过 webpack 配置项的 Externals 来实现。

Externals 配置项告诉 Webpack 项目中使用了哪些不用被打包的模块，相关配置如下：

```webpack.config.js
    module.exports={
        ...
        externals:/^(react|babel-runtime)/,
    }
```

开启上述配置后，在打包出来的代码还是存在导入 react 或 babel-runtime 的代码，但它们的 react 或 babel-runtime 内容并不会包含在打出的包里。
这样做是为了确保代码的正确性的前提下，减小应用包的体积。

在实际的开发上，我们远远不止要对 react 和 babel-runtime 做这样的处理，而是要对正在开发依赖的包做这样的处理，因为它们也有可能被引用你的包的项目所依赖，这样就会造成使用你的模块的应用构建的包的体积变大，因为有些依赖包被重复打包了

#### 测试 NPM 包

日常使用第三方模块，我们都是将 npm 线上的包通过 npm install 到我们的项目里，但这并不好调试，每次都要发布到 npm 上，然后 update 本地的依赖包，然后测试。
我们可以通过 npm link 来再本地测试好，然后在发布到 npm 上。

#### 发布 NPM 包

在发布我们开发好的模块到 npm 仓库前，我们要保证 package.json 文件的正确性。比如我们生成的 ES5 文件到 lib 文件夹下，那么 main 字段对应的就是 lib/index.js:

```package.json
    {
        main:"lib/index.js",
        jsnext:main:"src/index.js"
    }
```

jsnext:main 字段用于指出采用 ES6 编写的源码的入口文件，这样做的目的是为了方便实现 Tree Shaking

在项目的根目录下执行 npm publish (确保已经 npm login 过了)

思考：webpack 在构建过程中，将多个模块合并成一个 chunk 输出，这对我们构建一个完整不可分割的库是一致的，但如果想 Lodash 这样工具库，我们可能只想使用 lodash 的某个功能，比如 cloneDeep，但这个时候 lodash 是一个完整的模块显然是不合适的，又如一些 UI 库。是否可以使发布后的代码结构跟源码保持一致呢？

ant-design 通过按需加载
