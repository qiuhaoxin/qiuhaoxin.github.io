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
