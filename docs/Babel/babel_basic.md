### Babel <!-- {docsify-ignore} -->

#### 介绍

- 1、Babel 是 JS 的编译器，作用就是负责把未来的 ECMAScript 语法标准(比如 ES6+)翻译成现在大部分浏览器支持的 ECMAScript 语法(比如 ES5)。
  这其中包括了新语法的转换(transform)和缺失特性的修补(polyfill)
  例如：

```Javascript
    const add=(a,b)=>{
        return a + b;
    }
    //可以转换成
    var add=function(a,b){
        return a + b;
    }
```

- 2、Babel 是构建于插件之上的，通过一系列的插件构成了一个个转换通道，我们可以通过使用或创建一个预设(preset)来使用一组
  插件，编码编写过多的代码来引入插件。
- 3、Babel 只负责转换 JS，其他的一些框架或者类型检测语言需要额外的一些插件比如：
  @babel/preset-react、@babel/preset-flow、@babel/preset-typescript 等。

#### 使用

我们开发所需要的的 babel 的模块都是以独立 npm 包来发布的。可以安装以下 npm 包：

- @babel/core: Babel 核心库

- @babel/cli: CLI 终端命令行工具

- 预设(preset)与插件(plugin)
  代码转换功能是已插件的形式来进行的，一个插件就是一个 NodeJS 格式的模块，用于指导 Babel 如何对代码进行转换。例如@babel/plugin-transform-arrow-functions 之类的官方插件
  ```
    ./node_modules/.bin/babel src --out-dir dist --plugins=@babel/plugin-transform-arrow-functions
  ```
  根据职责单一原则，一个 Babel 插件只负责转换一种语法规则，ES2015+又那么多的新语法特性，每种语法特性都要添加一个插件显然是不好的，所有也有了预设(即 Preset),可以预设一组插件,我们可以这样做：
  ```Javascript
    npx babel src --out-dir dist --presets=@babel/preset-env
  ```
- 配置文件
  上面都是通过命令行来对 js 进行编译，但 preset 也是可以设置参数的，比如@babel/preset-env 的作用是将最新的语法特性和缺失特性修补到项目中，这样无论项目有没有使用的特性也一股脑注入到代码中，造成了代码的膨胀和冗余。我们可以通过配置文件给预设传参。
  可以通过 babel.config.json(v7.8.0 或更高的版本)或者 babel.config.js 文件来配置 babel 的预设和插件

  ```JSON
    "presets":[
        [
            "@babel/preset-env",
            {
                "targets":{
                    "edge":"17",
                    "chrome":"60",
                    "firefox":"30",
                    "safari":"11.1"
                }
            }
        ]
    ]
  ```

  在 babel 工作时就会默认读取项目根目录下的 babel 配置文件，这样 env 预设有了参数，就会针对项目所支持的浏览器的版本来对语法和缺失进行修补。

- Polyfill
  针对 Babel 7.4.0 开始，@babel/polyfill 这个包已经放弃使用了,替代的是用 core-js/stable(做 ECMAScript 语法转换)和 regenerator-runtime/runtime (需要使用转译后的生成器函数)
  ```Javascript
    import "core-js/stable";
    import "regenerator-runtime/runtime";
  ```
  其实@babel/polyfill 也是包含 core-js 和 自定义的 regenerator runtime 的。添加上述代码我们就可以使用 Promise、WeekSet 这样新增的内置组件和 Array.from、Object.assign 这样的类静态方法、Array.prototype.includes 这样的实例方法了。还可以使用 generator(添加了 regenerator 的插件)。为了添加上述的功能，polyfill 将添加到全局范围(global scope)和类似 String 这种原生原型中。但这样太多了，比如我不需要 Array.prototype.includes 这样的方法，我们可以使用 transform runtime 插件而不是对全局范围(global scope)造成污染的@babel/polyfill.如果我们知道我们要使用哪些特性，也可以直接从 core-js 获取它们。比如
  ```Javascript
    require("core-js/modules/es.promise.finally");
  ```
  另外：@babel/preset-env 也提供有"useBuiltIns"参数来进行优化
