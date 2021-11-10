## 前端中的几种观察者模式

### ResizeObserver

允许我们观察 DOM 元素的内容矩阵大小(width、height)的变化，并做出相应的响应。它就像给元素添加 document.onresize 或 window.resize 事件。不调整视窗大小而只是改变元素的大小，这是很有用的。例如添加新的子元素，将元素的 display 设置为 none 或类似可以改变元素大小的事件，事实上，它只关心内容框(Content Box)变化。比如下面行为会触发 ResizeObserver 的行为：

- 、当观察到的元素被插入或从 DOM 中删除是，观察将会触发
- 、当观察到的元素 display 值为 none 时，观察会触发
- 、观察不会对未替换的内联元素(non-replaced inline element)触发
- 、观察不会由 CSS 的 transform 触发
- 如果元素有显示，而且元素大小不是 0,0 观察将会触发

使用：

```Javascript
    const observer=new ResizeObserver(handler);

    function handler(entries){
        entries.forEach(entry=>{
            const size=entry.contentRect;
            const {width,height}=contentRect;
        })
    }

    const child=document.querySelector('.box');

    observer.observe(child);
```

### PerformanceObserver

主要用于观察性能时间轴(Performance Timeline),并在浏览器记录时通知新的性能条目。它可以用来度量浏览器和 Node.js 应用程序中的某些性能指标。在浏览器中，我们可以通过 window 属性 window.PerformanceObserver 来获取，在 Node.js 应用程序中需要 perf_hooks 获得性能对象。比如：const {performance}=require('perf_hooks')。它在下列情况下是有用的：

- 测量请求(Request)和响应(Response)之间的处理时间。(浏览器)
- 在从数据库中检索时计算持续时间(Node.js 应用程序)
- 抽象精确的时间信息，使用 Paint Timing API，比如 First Paint 或 First Contentful Paint 时间
- 使用"User Timing API"、"Navigation Timing API"、"Network Information API"、"Resource Timing API"和"Paint Timing API"访问性能指标。

使用：

- 、创建观察者

```Javascript
    const observer=new PerformanceObserver(logger);
    //浏览器端
    function logger(list,observer){
        const entries=list.getEntries();
        entries.forEach(entry=>{

        })
    }
    //服务端
    function getDataFromServer(){
        performance.mark('startWork');
        doWork();
        performance.mark('endWork');
        performance.measure('start to end','startWork','endWork');
        const measure=performance.getEntriesByName('start to end')[0];
    }
```

- 、定义要观察的目标对象

```Javascript
    /*
    * 可以通过PerformanceObserver.supportedEntryTypes 来查看支持的entryType类型
    * ['element', 'event', 'first-input', 'largest-contentful-paint', 'layout-shift', 'longtask', 'mark', 'measure', 'navigation', 'paint', 'resource']
    */
    PerformanceObserver.supportedEntryTypes
    const config={
        entryTypes:['mark','measure']
    }
    observer.observe(config);

    // 有效的entryTypes 值：
    // "mark":[USER TIMING]
    // "measure":[USER TIMING]
    // "navigation":[NAVIGATION-TIMING-2]
    // "resource":[RESOURCE-TIMING]
```

### MutationObserver

用于观察 DOM 元素的变化。可以观察到在父节点中添加或删除子节点、属性值或数据内容的变化。在 MutationObserver 之前 DOM 更改事件一般都是用 Mutation 事件处理，比如 DOMAttrModified、DOMAttributeNameChanged 和 DOMNodeInserted，主要是性能原因。

使用：

- 创建观察者

```
    const observer=new MutationObserver(handler);

```

- 定义要观察的目标对象

```Javascript
    const parent=document.querySelector(".parent");
    let config={
        childList:true,//表示观察与子节点相关的变化
        attributes:true,//表示观察属性更改
        characterData:true,//表示观察目标元素的数据内容的变化
    }
    observer.observe(parent,config);
    /*
    * config:{
        attributes:true,
        attributeOldValue:true,
        characterData:true,
        characterDataOldValue:true,
        childList:true,
        subtree:true,
    }
    */
```

- 定义回调事件

```Javascript
    function handler(mutationRecords,observer){
        mutationRecords.forEach(mutationRecord=>{
            const type=mutationRecord.type;
            switch(type){
                case 'childList':

                break;
                case 'attribute':

                break;
            }
        })
    }
```

### IntersectionObserver

用于观察两个 HTML DOM 元素之间的交集。当 DOM 进入或离开可见的视窗(Visible Viewport)时，在 DOM 中观察一个元素是很有用的，可以在以下的场景使用该 API：

- 1、当一个元素在视窗中可见时，延迟加载图像或其他资源
- 2、识别广告的可见性并计算广告收入
- 3、实现网站的无限滚动，当用户向下滚动页面时，不必要浏览不同的页面
- 4、当一个元素在视窗中时，加载和自动播放视频或动画

对比以前做元素的相交检测一般都是基于元素事件和监听元素的 getBoundingClientRect()方法来获取元素的边界信息。但元素事件和 Element.getBoundingClientRect()方法都是运行在主线程上的，有可能会频繁触发，导致性能问题。而 Intersection API，浏览器会自行做优化

注意 Intersection Observer API 无法提供重叠的像素个数或者具体哪个像素重叠，他的更常见的使用方式是——当两个元素相交比例在 N% 左右时，触发回调，以执行某些逻辑。

使用方法的步骤：

- 1、创建观察者

```Javascript
    //语法描述
    const observer=new IntersectionObserver(handler,option)

    /*
    *
    handler是回调函数格式为function(entities){
        entities.forEach(entity=>{
            const {}=entity;
        })
    }
    */
    /*
     option:{
        root:所监听对象的具体祖先元素，如果未传入或为null，则默认使用顶级文档的视窗
        rootMargin:计算交叉时添加到根(root)边界盒bounding box (en-US)的矩形偏移量， 可以有效的缩小或扩大根的判定范围从而满足计算需要。此属性返回的值可能与调用构造函数时指定的值不同，因此可能需要更改该值，以匹配内部要求。所有的偏移量均可用像素(pixel)(px)或百分比(percentage)(%)来表达, 默认值为"0px 0px 0px 0px"。
        thresholds:一个包含阈值的列表, 按升序排列, 列表中的每个阈值都是监听对象的交叉区域与边界区域的比率。当监听对象的任何阈值被越过时，都会生成一个通知(Notification)。如果构造器未传入值, 则默认值为0。当值为1时意味着目标元素完全出现在根元素中。可以是一个Number值或Number数组
     }
    */

    /*
    方法：
    disconnect:使IntersectionObserver对象停止监听工作
    observe():使IntersectionObserver 开始监听一个目标根元素
    takeRecords():返回所有观察目标的IntersectionObserverEntry对象数组。
    unobserve():使IntersectionObserver停止监听特定目标元素
    */
```

- 2、定义要观察的目标对象

```Javascript
    const target=document.querySelector('.target');
    observer.observe(target);
```

- 3、定义回调事件
  具体回调事件如上的 handler。
