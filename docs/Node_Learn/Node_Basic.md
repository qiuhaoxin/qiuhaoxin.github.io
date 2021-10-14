## Node 基础及核心模块学习 <!-- {docsify-ignore} -->

### 概述

Node 特点：事件驱动、异步处理、非阻塞 I/O

### Stream

#### 流是什么

流跟其他数据一样，也是一系列的数据，不同的是 stream 无法一次性全部可用，而且它们不需要跟内存完全合槽，在处理大量数据或者一个一次性只给一部分的
数据源的时候显得特别有用。另外也可以通过管道来组合一些命令。

该模块是 nodejs 的很重要的模块，在 nodejs 中很多模块都是 stream 的实例，比如：http.clientRequest、process.stdout

#### 内置流的分类 <!-- {docsify-ignore} -->

- 1、Readable：用来读取数据 比如 fs.createReadStream();
- 2、Writable：用来写数据 比如 fs.createWriteStream();
- 3、Duplex：可读 + 可写 比如 net.Socket
- 4、Transform：在读写的过程中，可以对数据进行修改 比如 zlib.createDeflate() (数据压缩/解压)

#### Readable Stream 可读流 <!-- {docsify-ignore} -->

Events: data、end、error、close、readable
常用的事件有：
data：当流中传出一块数据给消费者的时候会触发
end：当没有更多数据时触发
Functions：pipe(),unpipe(),read(),unshift(),resume()
pause(),isPaused(),setEncoding()
以下是 nodejs 中常见的 ReadStream 的实例：

可读流有两种模式：
暂停(Paused)模式:默认情况下，可读流以暂停模式启动，我们可以通过 read()函数来按需读取。
流动(Flowing)模式：这种模式下，数据是延绵不断的，需要用事件监听消耗数据，因为在流动模式下，如果不消耗数据，数据是会丢失的。
可以通过 pause()和 resume()切换上述两种模式。

```Html
    1、http.IncomingMessage in Server、Server Response in client
    2、fs.createReadStream
    3、process.stdin
    4、child_process stderr and stdout
```

以下的一些例子：

```Javascript
    const fs=require('fs');
    const readStream=fs.createReadStream('./sample.txt');
    let data='';
    readStream.setEncoding('utf-8');
    readStream.on('data',function(chunk){
        data+=chunk;
    });
    readStream.on('end',function(){
        console.log("输出的内容是：%s",data);
        //输出的内容是：我是邱浩新
    })
```

有时我们也会利用流来减少内存的消耗：

```Javascript
    // 这样我们就不用先把index.html文件先读到内存，然后在从内存读出来响应请求
    //特别是当文件比较大的时候，会更明显
    const http=require('http');
    const fs=require('fs');
    http.createServer((req,res)=>{
        if(req.url=='/index.html'){
            const f=fs.createReadStream('./public/index.html');
            f.pipe(res);
            res.end();
        }
    })
```

#### Writable Stream

Events:drain、finish、error、close、pipe/unpipe
常用事件：
drain:触发后表示可写流可以写入数据
finish:触发后表示数据已经写入到下层系统了

Functions：write()、end()、cork()、uncork()、
setDefaultEncoding()
写文件

```Javascript
    const fs=require('fs');
    const content="hello world!";
    const ws=fs.createWriteStream('./test.txt');
    ws.write(content);
    ws.end();
```

#### Duplex Stream

这个最为常见的是 net.Socket

```Javascript
    //服务端
    const net=require('net');
    var opt={
        host:'localhost',
        port:4001,
    }
    const server=net.Socket(opt);
```

#### Transform Stream
