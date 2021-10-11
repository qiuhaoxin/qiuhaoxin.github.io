# 关于 Axios 使用与源码学习

## 介绍

Axios 是一个基于 Promise API 的，可用于浏览器和 Node 端的 HTTP 库。

- 从浏览器创建 XMLHttpRequest
- 从 Node 创建 http.request
- 基于 Promise API
- 设置拦截器拦截请求和响应
- 在处理请求和响应前装换数据
- 取消请求
- 自动装换 JSON 数据
- 客户端支持防御 XSRF

## 安装

使用 npm：

```Javascript
    npm i axios -S
```

使用 CDN：

```Javascript
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>

```

## 使用

- 因为导出的 axios 作为一个包装 Axios 实例的一个函数,可用直接通过传入请求参数调用 axios ：

```Javascript
    const axios=require('axios');
    axios({

    }).then(res=>{

    }).catch(err=>{

    });
    //或
    axios(url,{

    }).then(res=>{

    }).catch(err=>{

    })
```

- 可用基于 axios 对象上 extend 的请求方法调用

```Javascript
    const axios=require('axios');
    axios.get(url,options).then(res=>{});
    axios.post(url,{}).then(res=>{})
```

- 直接使用 axios.request()方法调用，request 是 Axios 上的原型方法，也是其他所有封装方法的底层：

```Javascript
    const axios=require('axios');
    axios.request({

    })
    //或
    axios.request(url,{})
```

- 执行并发请求：

```Javascript
    const axios=require('axios');
    function getUserList(params){
        return axios.get(params.url);
    }
    function getUserAccount(params){
        return axios.get(params.url)
    }
    axios.all([getUserList(),getUserAccount()]).then(function(result){
        const userList=result[0];
        const userAccount=result[1];
        ...
    })
```

- 利用 axios 提供的 create 方法，创建实例（本质返回的还是基于 Axios 实例的包装函数）,可以基于该实例调用各种请求方法：

```Javascript
    const axios=require('axios');
    const instance=axios.create(selfConfig);
    instance.get({});
    instance.post({});
```

设置拦截器：

```Javascript
    const axios=require('axios');
    //设置请求前的拦截器，对config进行修改并返回config
    axios.interceptors.request.use(function(config){
        ...
        return config;
    })
    //设置响应后的拦截器，对响应内容进行修改并返回
    axios.interceptors.response.use(function(response){
        ...
        return response;
    })
```

修改默认配置：
axios 库本身内置了一部分 config，比如通用的 http 首部字段:Accept,可以通过：

```Javascript
    const axios=require('axios');
    axios.defaults.headers.common['Accept']="text/plain";
```

贴上请求的配置项，在项目应用中可以根据实际进行修改：

```Javascript
{
   // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // default

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data, headers) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

   // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

   // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // default

  // `adapter` 允许自定义处理请求，以使测试更轻松
  // 返回一个 promise 并应用一个有效的响应 (查阅 [response docs](#response-api)).
  adapter: function (config) {
    /* ... */
  },

 // `auth` 表示应该使用 HTTP 基础验证，并提供凭据
  // 这将设置一个 `Authorization` 头，覆写掉现有的任意使用 `headers` 设置的自定义 `Authorization`头
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

   // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default

   // `xsrfCookieName` 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default

  // `xsrfHeaderName` is the name of the http header that carries the xsrf token value
  xsrfHeaderName: 'X-XSRF-TOKEN', // default

   // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

   // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,

  // `validateStatus` 定义对于给定的HTTP 响应状态码是 resolve 或 reject  promise 。如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，promise 将被 resolve; 否则，promise 将被 rejecte
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  // `maxRedirects` 定义在 node.js 中 follow 的最大重定向数目
  // 如果设置为0，将不会 follow 任何重定向
  maxRedirects: 5, // default

  // `socketPath` defines a UNIX Socket to be used in node.js.
  // e.g. '/var/run/docker.sock' to send requests to the docker daemon.
  // Only either `socketPath` or `proxy` can be specified.
  // If both are specified, `socketPath` is used.
  socketPath: null, // default

  // `httpAgent` 和 `httpsAgent` 分别在 node.js 中用于定义在执行 http 和 https 时使用的自定义代理。允许像这样配置选项：
  // `keepAlive` 默认没有启用
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // 'proxy' 定义代理服务器的主机名称和端口
  // `auth` 表示 HTTP 基础验证应当用于连接代理，并提供凭据
  // 这将会设置一个 `Proxy-Authorization` 头，覆写掉已有的通过使用 `header` 设置的自定义 `Proxy-Authorization` 头。
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // `cancelToken` 指定用于取消请求的 cancel token
  // （查看后面的 Cancellation 这节了解更多）
  cancelToken: new CancelToken(function (cancel) {
  })
}
```

## 源码

在了解 axios 的特性和使用后（无论是什么库，在看源码之前建议务必了解库本身的特性和如何使用，这样我们在阅读的时候才不至于被细节困扰），我们现在正式进入源码的学习
另外，因为 axios 只是根据宿主环境分别对浏览器的 XMLHttpRequest 和 Node 的 http 模块的 request 进行封装，涉及到具体的知识点，可以翻翻上面的 xhr 和 http.request 复习相关知识点。

### 代码结构

从 github 把代码 clone 下来到本地后，打开代码目录，可以发现源码主要在 lib 文件夹下，如图：
![axios源码结构图](./imgs/source_code.png)

- adapters 文件夹 主要是适配器，浏览器下是 XMLHttpRequest，而在 Node 下是 http 模块的 request,可以通过修改配置项的 adapter 自定义请求
- cancel 文件夹是取消请求的相关文件
- core 文件夹 是库的核心，主要包括 Axios 类、拦截器类、mergeConfig 工具等
- helpers 文件夹主要提供 Http 首部字段处理、url 处理、校验等辅助方法
- axios.js 是入口文件，导出变量、并基于 Axios 进一步封装、扩展
- default.js 是初始化 Axios 实例默认的配置项
- utils.js 提供一些基础的方法，比如数据类型的判断、封装 forEach 等

### 入口文件

我们先看看 lib/axios.js,在 package.json 中 main 字段定义库的入口为 index.js，但由于在 index.js 中仅有一行代码 require('./lib/axios.js') 引用了 axios.js,故这里称 axios.js 为入口文件，真正涉及到源码的地方：
![lib/axios.js](./imgs/axios_begin.png)

看上述代码很明显的，用 CJS 模块语法导出了一个 axios 对象，为了兼容 TypeScript，还具名 default 做了兼容。我们看得出来，axios 对象上是挂载了 all、spread 等方法，也挂载了 CancelToken、Cancel、isCancel 这些属性，为用户提供了快捷的方法使用（其实它们都是函数的类方法）
