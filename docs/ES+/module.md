### 前端模块化

前端经历了一个凌乱的时代，远古时代，前端是没有模块管理的，常规加载的模式如下：

```Html
    <script src="a.js" type="text/javascript">
    <script src="b.js" type="text/javascript">
    <script src="c.js" type="text/javascript">
```

这样导致很多问题：

- 1、应用大了，逻辑复杂了，应用的可维护性，可读性都很难保证
- 2、这样很难去做按需加载，打开一个页面也要加载上面全部脚本
- 3、脚本的版本很难管理
- 4、脚本的依赖关系很难管理

#### AMD & CMD

这是社区为浏览器环境提出的模块加载方案，AMD 全称为异步模块定义,
采用异步的方法去加载依赖的模块，最具代表性的实现是 requirejs。
语法如下：

```Javascript
    // 定义一个模块
    define('module',['dep'],function(de){
        return exports;
    })
    // 导入和使用
    require(['module'],function(module){

    });
```

AMD 的优点在于：

- 可以不在转换代码的情况下直接在浏览器中运行
- 可异步加载依赖
- 可并行加载多个依赖
- 代码可运行在浏览器和 Node 环境下

缺点：
在于 JavaScript 运行环境没有原生支持 AMD，需要先导入实现了 AMD 的库后才能正常使用。

#### ES Module

ES6 在语言层面上为我们引入了标准的模块化，这个未来将会被浏览器和 Node 所采用的模块化标准。
特征：

- 1、严格模式
  在 ES Module 里，严格模式将会默认开启

  ```Javascript
        //严格模式禁止一些不良的语法行为
    1、变量不能未声明使用
    2、函数参数不能有相同变量名称
    3、with 关键字不允许使用
    4、delete 不能删除的属性会抛出异常
    5、delete prop is a syntax error,instead of assuming delete global[prop]
    6、不再支持arguments.callee，会抛出TypeError异常
    7、不再支持arguments.caller,会抛出TypeError异常
    8、Context passed as this in method invocations is not “boxed” (forced) into becoming an Object
    9、No longer able to use fn.caller and fn.arguments to access the JavaScript stack
    10、Reserved words (e.g protected, static, interface, etc) cannot be bound

  ```

  模块内的变量具有模块作用域，模块定义的变量或函数只能在模块内使用，其他模块要使用只能通过 export 关键字导出相应的变量。
  export 模块导出类型

- 1、默认导出
- 2、具名导出
  语法如下：

```Javascript
    export default 6;
    export var foo=6;

    var foo="ponyfoo";
    var bar="baz";
    export {foo,bar}

    export {foo as ponyfoo}

    export {foo as default,bar};
```

注意：export 只能放置在模块文件的顶部，不能放置代码块内。这个限制更方便解析器去解析代码

Bindings,Not Values
我们要意识到，ES Module 的导出是变量的绑定关系，而不是值或引用。这意味着模块里的变量发生变化，那么导出的变量也跟着变化，而不是像 CommonJS 那样导出的是变量的副本或引用

```Javascript
    export var foo="bar";
    setTimeout(()=>foo="baz",500)
```

#### ES Module 在浏览器

ES Module 可以在以下浏览器版本原生支持了

Safari 10.1
Chrome 61
Firefox 60
Edge 16

语法：

```Javascript
    <script type="module">
        import {addTextToBody} from './utils.mjs'
        addTextToBody('Modules are pretty cool');
    </script>
```

```Javascript
    //utils.mjs
    export function addTextToBody(text){
        const div=document.createElement('div');
        div.textContent=text;
        document.body.appendChild(div);
    }
```

跟常规的脚本相比，module script 多了一个 type="module" 的属性。这样浏览器就会把行内脚本或外部脚本当做一个 ECMAScript 的模块。

- 裸导入目前还不支持

```Javascript
    import {foo} from 'https://xxx.com/utils/bar.mjs';
    import {foo} from './utils/bar.mjs';
    import {foo} from './bar.mjs';
    import {foo} from '../bar.mjs';

    //Not supported;
    import {foo} from 'bar.mjs';
    import {foo} from 'utils/bar.mjs';
```

有效的模块说明符必须是以下几种：

- 一个完整的 url
- 以/开头
- 以./开头
- 以../开头
  其他的是未来的保留关键字，例如导入内置模块。

用 nomodule 标签特性来做向后兼容性：

```Javascript
    <script type="module" src="module.mjs"></script>
    <script nomodule src="fallback.js"></script>
```

说明：支持 type="module" 的浏览器会自动忽略 nomodule 的属性，这样一来那些不支持 module 的浏览器就会执行 nomodule 的脚本

- module script 默认 defer

```Javascript
    <script type="module" src="1.mjs"></script>
    <script src="2.js"></script>
    <script defer src="3.js"></script>
```

常规的 js 脚本在加载解析阶段会阻塞 HTML 的解析，我们如果确保在 HTML 解析时脚本不会对 dom 作出修改，可以在脚本添加 defer 属性来异步加载，但脚本加载完后也会在 HTML 解析完后执行，而且是保持脚本加载顺序执行。Module script 默认有 defer 的行为。

所以上述的脚本执行顺序： 2.js,1.mjs,3.js

- 同理，行内的 module script 也是 deferred 的。

```Javascript
    <script type="module">
        addTextToBody("Inline module executed")
    </script>
    <script src="1.js"></script>
    <script defer>
        addTextToBody("Inline script executed")
    </script>
    <script defer src="2.js"></script>
```

脚本执行顺序：1.js,inline script,inline module,2.js。

行内的脚本是忽略 defer 属性的，而行内的 module 脚本是默认 deferred 的（无论是否有导入 import）。

- module script with async

```Javascript
    <script async type="module">
        import {addTextToBody} from './utils.mjs';
        addTextToBody("Inline module executed.")
    </script>
    <script async type="module" src="1.mjs"></script>
```

执行结果：下载快的脚本将优先执行，上面的行内脚本会等待依赖的 utils.mjs 脚本加载完毕后执行，而 1.mjs 会等待自身和依赖加载完后才执行。

常规的脚本，async 会导致脚本的加载不会影响 HTML 的 parse，下载完后会尽快执行，这样会导致脚本的执行并不会按脚本的顺序，async 属性对 inline module 同样有效。

- modules 只执行一次

```Javascript
    <!-- 1.mjs only executes once -->
    <script type="module" src="1.mjs">
    </script>
    <script type="module" src="1.mjs"></script>
    <script type="module">
        import './1.mjs'
    </script>

    <!-- 常规的脚本加载多少次就执行多少次 -->
    <script src="2.js"></script>
    <script src="2.js"></script>

```

- Always CORS

```Javascript
//这个脚本不会执行，因为它的CORS 检查失败了
    <script type="module"
    src="https://.../no-cors"></script>
    //这个脚本也不会执行，因为它的一个导入CORS检查失败
    <script type="module">
        import 'https://....now.sh/no-cors';
        addTextToBody("This will not execute!");
    <script>
    //这个脚本会执行，因为它通过了CORS检查
    <script type="module" src="https://....now.sh/cors">

    </script>
```

跟常规脚本不一样，module scripts(and their imports) are fetched with CORS.
这就意味这 cross-origin module scripts 必须返回有效的 CORS 头部字段，例如：
Access-Control-Allow-Origin:\*

- Credentials by default
  如果是请求是同源的，大多数基于 CORS-based APIs 都会发送 credentials(cookies 等)，但 fetch API 和 module scripts 并不会。但现在 fetch API 和 module scripts 也会发送 credentials 了。
  为了兼容不同浏览器、不同版本、不同时期发送 credentials。我们可以用 crossorigin 属性，会发送 credentials 去同源的请求，但如果是跨源

```Javascript
    <!-- Fetched with credentials(cookie etc) -->
    <script src="1.js"></script>
    <!-- Fetched with credentials,except in old browsers that follow the old spec -->
    <script type="module" src="1.mjs"></script>
    <!-- Fetched with credentials,in browsers that follow the old & new spec -->
    <script type="module" crossorigin src="1.mjs"></script>
    <!-- Fetched without credentials -->
    <script type="module" crossorigin src="https://other-origin/1.mjs"></script>
    <script type="module" crossorigin="use-credentials" src="https://other-origin/1.mjs"></script>
```

- Mime-types

跟常规脚本不一样，modules scripts 必须是 text/javascript 的 mime type 类型，否则不会执行。

[推荐阅读](https://jakearchibald.com/2017/es-modules-in-browsers/#always-cors)
