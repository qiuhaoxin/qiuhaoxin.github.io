## Webpack Loader 介绍及使用 <!-- {docsify-ignore} -->

### loader 作用

webpack 只能处理原生的 js，其他模块资源需要通过 loader 来解析，loader 就像一个具有文件转换功能的翻译员，把 webpack 不认识的资源翻译成认识的资源

### 常见的 Loader 及使用

loader 的配置在 webpack.config.js 的 module.rules 字段，例如：

```JS
    module.exports={
        ...
        module:{
            rules:[
                {
                    test:/.css$/,
                    use:[
                        'style-loader',
                        'css-loader?minimize'
                    ]
                }
            ]
        }
    }
```

这相当于告诉 Webpack，如果再解析文件的过程中，遇到由.css 结尾的文件，先用 css-loader 去解析，然后把解析结果给 style-loader,由它去做剩下的工作，并把结果返回给 Webpack。
注意：

- 1、use 属性是一个有 loader 构成的数组，执行的顺序是由右到左，由下到上
- 2、如果需要携带参数，可以通过 query string 的方式传入参数，比如上例的 minimize，具体 loader 的参数要查询 loader 官方文档，还支持 object 的形式配置，比如 options:{minimize:true}

### 如何编写自定义的 Loader

### 一个常用 Loader 源码剖析
