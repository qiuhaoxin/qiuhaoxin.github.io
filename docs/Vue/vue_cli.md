## Vue Cli 源码解读 <!-- {docsify-ignore} -->

### 介绍

Vue CLI 是一个 Vue 的脚手架。

### 提前准备

Vue Cli 工具用了几个挺重要的库，先做个简单介绍：

- 1、Inquirer
  用官方文档介绍的话，这是一个 A collection of common interactive command line user interfaces 即
  A collection of common interactive command line user interfaces
  通用交互式命令行用户界面的集合

它提供以下常用的方法：

```Javascript
    1、inquirer.prompt(questions, answers) -> promise

    questions 是 Array<Question>,是问题集的组成,在CLI生成的流程中，when的属性是用来根据之前的answers来控制某个问题是否要问的
    answers 是 Answers 对象
    返回一个Promise
    2、inquirer.registerPrompt(name,prompt)
    3、inquirer.createPromptModule() -> prompt function
    Question:{
        type:String,//input, number, confirm, list, rawlist, expand, checkbox, password, editor
        name:String,
        message:String,
        default:String,
        choices:Array|Function,

    }
    Answer:{
        key:String, //Question 的name值
        value:, //取决Question的type input、number为用户输入，如果是confirm为boolean，如果是list、rawlist 为用户选择的值
    }
```

- 2、joi
  lets you describe your data using a simple, intuitive, and readable language(允许你用简单、直观、可读的语言描述你的数据).

  ```Javascript

  ```

- 3、commander

- 4、execa
  这个库主要是用来执行命令脚本的，是对 child_process 方法的提升：
- 1、支持 Promise 接口
- 2、从输出中剥离最后的换行符，不用显示调用 stdout.trim()
- 3、支持 shebang 二进制文件跨平台
- 4、window 系统的支持
- 5、更大的 buffer size，100MB 而不是 200kb
- 6、

#### Vue CLI 提供了多个指令

- 1、create
  语法：

```Javascript
   vue create projectName [options]
```

创建一个新的 vue 项目，并附带一些选项来配置项目

- 2、add
  在项目里添加一个插件，并调用插件的 generator

- 3、invoke 方法
  在项目里触发插件的 generator 方法

- 4、inspect 方法
  查看项目的 webpack 配置项，这个配置是基于@vue/cli-plugin-service 插件的

- 5、serve 方法
  相当于调用 npm run serve

- 6、build 方法
  想当于调用 npm run build

- 7、ui 启动 vue cli 的图形管理界面

- 8、config
  查看和修改配置

- 9、init 根据远程模板来初始化项目 依赖@vue/cli-init

- 10、upgrade 升级更新 vue cli service/plugins

我们下面主要对几个重要的指令的源码进行阅读：

每个指令都有选项，作者基于 commander 这个库来处理指令和选项

```Javascript
    const program = require('commander')
    program
    .command('directive [options]')
    .option('xxx <options>')
    .option('xxx [options]')
    .action((value,options)=>{
      // do something
    })
    program.parse(process.argv)
```

上面是 commander 的语法，定义了一个指令和指令对应的参数项，最后调用 program.parse(process.argv) 来处理参数，并在 action 的回调里传入参数。

#### create 指令源码阅读
