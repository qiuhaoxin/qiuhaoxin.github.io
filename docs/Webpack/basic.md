## Webpack 基本概念与使用 <!-- {docsify-ignore} -->

### 扩展

为什么会有构建？因为很多新的框架(如 React、Vue)、下一代语法标准(ES6+)、CSS 预处理器(LESS、SCSS 等)、不同浏览器等原因无法直接在宿主环境运行，必须要通过转换工具转换后才能正常运行，而构建就是做这些事情，把源代码转换成发布到线上的可执行 JavaScript、CSS、HTML 代码，具体包括以下内容：

- 代码转换：TypeScript 编译成 Javascript、SCSS 编译成 CSS 等
- 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片等
- 代码分割：提前多个页面的公共代码、提取首屏不需要执行部分的代码，让它们异步加载
- 模块合并：在采用模块化的项目里会有很多模块和文件，需要构建功能把模块分类合并成一个文件
- 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器
- 代码校验：在代码呗提交到仓库钱需要校验代码是否符合规范，以及单元测试是否通过
- 自动发布：更新完成代码后，自动构建出线上发布代码并传输给发布系统。

总的来说，构建是自动化、工程化在前端的体现，把一系列的流程用代码去实现，让代码来执行一系列复杂的流程。给前端注入了更多的活力，解放了生产力。
历史上大多数的构建工具都是通过 NodeJs 来完成的，下面来介绍一下构建工具：

- 1、Npm Script
  这是 npm 包管理工具内置的一个功能，允许在 package.json 里通过 scripts 字段定义要执行的任务：

```Javascript
    "scripts":{
        "start":"node ./start.js"
    }
```

通过命令行：npm run start 执行 start.js 的脚本
优点：内置的功能，无需额外的依赖
缺点：只提供了 pre、post 类型的钩子，不方便管理多个任务间的依赖

- Grunt

- Gulp
  Gulp 是一个基于流的自动化构建工具，除了可以管理、执行任务，还支持监听文件、读写文件，Gulp 被设计的非常简单，只通过以下 5 个方法就能胜任几乎所有的场景：

```Javascript
    1、通过Gulp.task 注册一个任务
    2、通过Gulp.run 执行任务
    3、通过Gulp.watch 监听文件变化
    4、通过Gulp.src 读取文件
    5、通过Gulp.dest 输出文件
```

Gulp 最大的特点就是引入了流来处理文件，还提供一系列插件来处理流，流可以在各个插件间传递，下面是一个例子：

```Javascript
    const Gulp=require('gulp');
    const jshint=require('gulp-jshint');
    const sass=require('gulp-sass');
    const concat=require('gulp-concat');
    const uglify=require('gulp-uglify');
    //编译scss任务
    gulp.task('sass',function(){
        gulp.src('./scss/*.scss')
        .pipe(sass())
        .pipe(gulp.dest('./css'));
    });
    gulp.task('scripts',function(){
        gulp.src('./js/*.js')
        .pipe(concat('all.js'))
        .pipe(uglify())
        .pipe(gulp.dest('./dest'));
    });
    gulp.task('watch',function(){
        gulp.watch('./scss/*.scss',['sass']);
        gulp.watch('./js/*.js',['scripts']);
    });
```

Gulp 的优点是好用又不失灵活，既可以单独完成构建也可以和其他工具搭配使用。缺点和 grunt 类似，集成度不高，要写很多配置才可以用，无法做到开箱即用。

- Fis3

- Webpack

- Rollup
  Rollup 是一个和 Webpack 类似但专注于 ES6 的模块打包工具。Rollup 的亮点在于能针对 ES6 源码进行优化比如(Tree Shaking、Scope Hoisting)来减小文件大小提升运行性能（但这些功能后来 webpack 也实现了）。
  和 Webpack 的区别

```Javascript
    1、Rollup是在Webpack流行后出来的替代品
    2、功能和生态链都不如Webpack完善，体验也不如，但配置和使用更加简单
    3、不支持Code Spliting，但好处在于打包出来的代码中么有Webpack那段模块的加载、执行和缓存的代码。
```

Rollup 在用于打包 Javascript 库时比 Webpack 更加有优势，因为其打包出来的包更小更快，但功能不够完善。

### Webpack 概念

Webpack 是一个打包模块化 Javascript 的工具，在 Webpack 里一切文件（包括 Javascript、CSS、SCSS、图片、模板等）皆模块，通过 Loader 来转换文件，通过 Plugin 来注入钩子，最后输出由多个模块组合成的文件。Webpack 专注于构建<strong>模块化</strong>项目。

好处：

- 专注于处理模块化的项目，能做到开箱即用
- 通过 Plugin 扩展，完整好用不失灵活
- 使用场景不仅限于 Web 开发
- 社区庞大活跃
  缺点：只能用于采用模块化开发

### Webpack 工作原理

Webpack 是一个打包模块化 Javascript 的工具，构建时，它会从配置文件的 entry 入口文件出发，识别模块中的模块导入语句(import\require),递归的寻找入口文件的所有依赖，把入口文件和其所有依赖打包到一个单独的文件中，最后输出

从 Webpack2 开始，已经内置了对 ES6、CommonJS、AMD 模块化语句的支持。

### Webpack 使用

- 、通过命令行 node_modules/.bin/webpack --config webpack.config.js

- 、通过 Npm Scripts

- 、通过 Node API 方式调用：

```Javascript
    const webpack=require('webpack');
    webpack({
        //webpack config
    },(err,stats)=>{
        if(err || stats.hasErrors()){
            //构建出错
        }
    })
```

上面代码只能构建一次，可以用监听模式来构建：

```Javascript
    const webpack=require('webpack');
    const webpackConfig=require('./webpack.config.js');
    const compiler=webpack(webpackConfig);
    const watching=compiler.watch({
        polling:300,

        aggregateTimeout:1000,
    },(err,stats)=>{

    })
    watching.close(()=>{

    })
```
