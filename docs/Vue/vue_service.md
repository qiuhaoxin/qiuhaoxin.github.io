### Vue-Cli-Service <!-- {docsify-ignore} -->

#### Vue-CLi-Service

Vue-Cli-Service 是 vue 提供的一个关于构建的 npm 包，如果是通过 vue-cli 脚手架创建的应用，可以自动安装该包。
这个包主要是以 webpack 为底层的构建工具进行了一些列的封装，并对外暴露了几个构建的方法。这个包可以跟 vue-cli 集成，也可以单独在项目安装使用

对外提供的构建方法：

- build：提供对 mode=production 线上应用构建的功能

```Javascript
    {
        "scripts":{
            "build":"vue-cli-service build"
        }
    }
```

- serve:提供应用的本地开发环境的功能

```Javascript
    {
        "scripts":{
            "dev":"vue-cli-service serve",
        }
    }
```

- inspect:查看 webpack 的配置参数，可以输出配置参数到一个文件

```Javascript
    {
        "scripts":{
            "getWebpackCfg":"vue-cli-service inspect",
        }
    }
```

- help:提供对 vue-cli-service 命令参数的帮助功能,可以带上其他命令

```Javascript
    npx vue-cli-service [options] help
    //
    npx vue-cli-service help build
```

- test:e2e / test:unit:提供端对端、单元测试

- lint 语法校验

#### 对外提供修改配置的接口

既然这是一个以 webpack 为底层进行封装的构建工具，那么必然提供了修改 webpack 的入口，可以通过以下方式来修改：

- 1、通过根目录下的 env、env.local、env.[mode] 来注入一下环境变量，环境变量在程序中可用，实现原理就是通过 webpack 的 DefinePlugin 来注入的

- 2、可以通过项目根目录下的 vue.config.js 来修改 webpack 的配置

#### 源码学习

Vue-Cli-Service 本质上是一个 node 写的一个 npm 工具，既然是 npm 工具，那么在项目描述文件就会有个 bin 的字段描述包的入口

```
    {
        "bin":{
            "vue-cli-service":"bin/vue-cli-service.js"
        }
    }
```

可以看出根目录下有个 bin 文件，里面的 vue-cli-service.js 就是在命令行或 scripts 脚本执行命令时的入口：

```
    npx vue-cli-service serve
    // 在scripts
    {
        "scripts":{
            "start":"vue-cli-service serve"
        }
    }
```

我们去 bin/vue-cli-service.js 看看入口做了些什么：

```Javascript
#!/usr/bin/env node   //指定用node来执行该脚本
// bin/vue-cli-service.js
const { semver, error } = require('@vue/cli-shared-utils')
const requiredVersion = require('../package.json').engines.node

//版本校验
if (!semver.satisfies(process.version, requiredVersion, { includePrerelease: true })) {
  error(
    `You are using Node ${process.version}, but vue-cli-service ` +
    `requires Node ${requiredVersion}.\nPlease upgrade your Node version.`
  )
  process.exit(1)
}

const Service = require('../lib/Service')
//实例化Service 对象
const service = new Service(process.env.VUE_CLI_CONTEXT || process.cwd())
// 这里提前命令行参数，第一个参数是node.exe程序路径，第二个是当前脚本的路径，第三个是脚本参数
const rawArgv = process.argv.slice(2)
//规范参数
const args = require('minimist')(rawArgv, {
  boolean: [
    // build
    // FIXME: --no-module, --no-unsafe-inline, no-clean, etc.
    'modern',
    'report',
    'report-json',
    'inline-vue',
    'watch',
    // serve
    'open',
    'copy',
    'https',
    // inspect
    'verbose'
  ]
})
const command = args._[0]  // 命令行的命令：build、serve、lint、test:unit 、test:e2e 、inspect
// 执行
service.run(command, args, rawArgv).catch(err => {
  error(err)
  process.exit(1)
})
```

看到入口文件就做了以下事情：

- 校验 node 的版本
- 实例化 Service 对象
- 提取参数
- 执行 service 对象的 run 方法，执行命令

看看在实例化 Service 时具体做了什么：

```Javascript lib/Service.js
    // class Service的构造函数
    constructor (context, { plugins, pkg, inlineOptions, useBuiltIn } = {}) {
        process.VUE_CLI_SERVICE = this
        this.initialized = false
        this.context = context  //设置项目根目录为上下文目录
        this.inlineOptions = inlineOptions
        this.webpackChainFns = []     // 收集改变webpack配置函数
        this.webpackRawConfigFns = []  //收集改变webpack配置
        this.devServerConfigFns = []   //webpack-dev-server 配置参数
        this.commands = {}    //命令的集合  {"build":{id:id,apply:fn},"serve":{id:id,apply:fn}}
        // Folder containing the target package.json for plugins
        this.pkgContext = context
        // package.json containing the plugins
        this.pkg = this.resolvePkg(pkg)
        // If there are inline plugins, they will be used instead of those
        // found in package.json.
        // When useBuiltIn === false, built-in plugins are disabled. This is mostly
        // for testing.
        this.plugins = this.resolvePlugins(plugins, useBuiltIn)  // 这个参数很重要，加载command所在的文件和webpack配置的文件
        // pluginsToSkip will be populated during run()
        this.pluginsToSkip = new Set()
        // resolve the default mode to use for each command
        // this is provided by plugins as module.exports.defaultModes
        // so we can get the information without actually applying the plugin.
        //初始化 modes
        this.modes = this.plugins.reduce((modes, { apply: { defaultModes } }) => {
        return Object.assign(modes, defaultModes)
        }, {})
    }
```

上面有两个很重要的事情，一个是调用了 resolvePlugins 来解析插件，第二则是初始化了 this.modes
看看 resolvePlugins 的代码：

```Javascript
    resolvePlugins (inlinePlugins, useBuiltIn) {
        //数组的一个迭代函数，返回{id:'built-in/commands/serve',apply:require('./commands/serve')}
        const idToPlugin = (id, absolutePath) => ({
            id: id.replace(/^.\//, 'built-in:'),
            apply: require(absolutePath || id)
        })

        let plugins
        // 对内置插件的执行文件执行map返回上面的一个数据结果，上面四个是vue-cli-service对外暴露的接口
        //下面5个则是webpack的一些配置参数
        const builtInPlugins = [
        './commands/serve',
        './commands/build',
        './commands/inspect',
        './commands/help',
        // config plugins are order sensitive
        './config/base',
        './config/assets',
        './config/css',
        './config/prod',
        './config/app'
        ].map((id) => idToPlugin(id))

        if (inlinePlugins) {
        plugins = useBuiltIn !== false
            ? builtInPlugins.concat(inlinePlugins)
            : inlinePlugins
        } else {
        //把配置文件中符合vue的插件提取出来，并加载生成 {id,apply}
        const projectPlugins = Object.keys(this.pkg.devDependencies || {})
            .concat(Object.keys(this.pkg.dependencies || {}))
            .filter(isPlugin)
            .map(id => {
            if (
                this.pkg.optionalDependencies &&
                id in this.pkg.optionalDependencies
            ) {
                let apply = loadModule(id, this.pkgContext)
                if (!apply) {
                warn(`Optional dependency ${id} is not installed.`)
                apply = () => {}
                }

                return { id, apply }
            } else {
                return idToPlugin(id, resolveModule(id, this.pkgContext))
            }
            })

        plugins = builtInPlugins.concat(projectPlugins)
        }

        // Local plugins   本地插件
        if (this.pkg.vuePlugins && this.pkg.vuePlugins.service) {
        const files = this.pkg.vuePlugins.service
        if (!Array.isArray(files)) {
            throw new Error(`Invalid type for option 'vuePlugins.service', expected 'array' but got ${typeof files}.`)
        }
        plugins = plugins.concat(files.map(file => ({
            id: `local:${file}`,
            apply: loadModule(`./${file}`, this.pkgContext)
        })))
        }
        debug('vue:plugins')(plugins)

        const orderedPlugins = sortPlugins(plugins)
        debug('vue:plugins-ordered')(orderedPlugins)

        return orderedPlugins
    }
```

实现了内置插件、配置符合 vue 格式插件和本地配置的插件的提取，实现了 modes 的初始化。

#### run 方法

执行 service.run()

```Javascript
async run (name, args = {}, rawArgv = []) {
    // resolve mode
    // prioritize inline --mode
    // fallback to resolved default modes from plugins or development if --watch is defined
    const mode = args.mode || (name === 'build' && args.watch ? 'development' : this.modes[name])

    // --skip-plugins arg may have plugins that should be skipped during init()
    this.setPluginsToSkip(args)

    // load env variables, load user config, apply plugins
    await this.init(mode)

    args._ = args._ || []
    let command = this.commands[name]
    if (!command && name) {
      error(`command "${name}" does not exist.`)
      process.exit(1)
    }
    if (!command || args.help || args.h) {
      command = this.commands.help
    } else {
      args._.shift() // remove command itself
      rawArgv.shift()
    }
    const { fn } = command
    return fn(args, rawArgv)
  }
```

run 方法确定了 mode，根据参数提取要跳过的插件，根据用户配置的 vue.config.js 和 env 文件来确定 webpack 配置的参数、chainFn 等。最后在
this.commands 提取相关的命令函数和参数，执行。

当 command name 为 serve 时执行 ./lib/commands/serve.js 的 serve 函数,首先通过 chainWebpack 添加修改 mode=development 下的参数设置 devtool 为 eval-cheap-module-source-map。接着执行 api.resolveWebpackConfig()获取 webpack 的配置。

#### 小技巧总结及借鉴

- 1、判断是 Promise

```Javascript
    const isPromise=(p)=> p && typeof p.then=='function'
```

- 2、判断网页是否运行在容器内：

```Javascript
    function checkInContainer(){

    }
```
