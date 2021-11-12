### 服务端渲染 SSR

#### 是否需要服务器渲染

要回答这个问题，首先我们要了解服务端渲染带来的好处和缺点：

好处：

- 更好的 SEO，因为搜索引擎爬虫工具可以抓取到完整的页面内容。
  首先我们先介绍一下传统的 SPA(Single Page Application)，它对 SEO 的支持并不好，单页面应用的 dom 由以前的 html 入口改成了 JS 入口，就是说 dom 的生成和页面内容的渲染更多依赖脚本而不是写在 html 来解析，这就要求我们的应用要先加载完依赖的脚本后，再去请求应用数据完成的页面渲染和展示。但很多的搜索引擎爬虫工具只支持同步的数据获取，如果一个应用在加载时给用户展示了一个 loading 的标记，背后通过 ajax 拉取服务端数据再做展示，拉取数据这个过程是异步的，搜索引擎爬虫工具并不会等待异步操作的结果返回，导致网站的排名很低，而服务端渲染会在服务端把业务数据和页面结合返回 html，这样就利于网站排名的提升。

- 更优的内容到达时间，特别是对于缓慢的网络和运行缓慢的设备
  无需等等所有的 js 脚本下载完才显示服务端渲染标志，用户能更快看到完整渲染的页面，也就是首屏时间会更短。

缺点：

- 开发条件所限，一些浏览器能使用的生命周期的钩子只能部分使用在服务端渲染中，另外一些库受限于服务器没有的变量(window、document 等)也不能使用

- 设计到构建和部署的更多要求，与可以部署在任何静态文件服务器上的完全静态单页面应用程序 (SPA) 不同，服务器渲染应用程序，需要处于 Node.js server 运行环境

- 加重服务器的负载。在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用 CPU 资源 (CPU-intensive - CPU 密集)，因此如果你预料在高流量环境 (high traffic) 下使用，请准备相应的服务器负载，并明智地采用缓存策略

总结：基于上述 SSR 的优点和缺点，如果你的应用场景需要更快的内容到达时间和更优的 SEO 体验，可以使用 SSR，但如果像我在一个 ERP 场景，对页面加载性能体验和 SEO 相对要求没有那么高的场景来说，这个就并不需要了。

#### vue 的 SSR

https://ssr.vuejs.org/zh/guide/#%E5%AE%89%E8%A3%85