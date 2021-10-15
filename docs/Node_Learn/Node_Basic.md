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

#### Transform Stream (转换流)

这个流中，运行对数据进行加工,Node 也内置了一些 Transform Stream 的模块，比如 zlib 和 crypto，下面结合 fs 模块对
指定文件进行压缩，并将压缩后的文件输出

```Javascript

    const fs=require('js');
    const zlib=require('zlib');

    const fileArg=process.argv[2];

    fs.createReadStream(fileArg)
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream(fileArg + 'gz'));

```

纯粹压缩，对用户体验不好，我们可以这样实现：

```Javascript
    const fs=require('js');
    const zlib=require('zlib');

    const fileArg=process.argv[2];

    fs.createReadStream(fileArg)
    .pipe(zlib.createGzip())
    .on('data',function(err){
        process.stdout.write('.');
    })
    .pipe(fs.createWriteStream(fileArg + 'gz'))
    .on('finish',()=>{
        console.log('Done')
    })
```

进一步的，我们还能利用下面自定义 Transform Stream 的知识了定义一个进度条相关的 Transform Stream：

```Javascript
    const {Transform}=require('stream');
    const fs=require('fs');
    const zlib=require('zlib');
    const progress=new Transform({
        transform(chunk,encoding,callback){
            process.stdout.write('.');
            callback(null,chunk);
        }
    })
    const fileArg=process.argv[2];

    fs.createReadStream(fileArg)
    .pipe(zlib.createGzip())
    .pipe(progress)
    .pipe(fs.createWriteStream(fileArg + 'gz'))
    .on('finish',()=>console.log('Done!'))

```

思考：

#### 说说 pipe()

语法：

```Javascript
    // 可读流.pipe(可写流)
    readableSrc.pipe(writableDest)

```

上面意思就是将数据源（也就是可读流）的输出，嫁接到数据目标（可写流）中去了。Duplex Stream 和 Transform Stream 可以既是数据源，也是数据目标
可以链式调用。

```Javascript
    //方法 pipe 返回的是数据目标
    b=a.pipe(b)
    // a 是可读流 b、c 是双工流 d 是可写流
    a.pipe(b).pipe(c).pipe(d)
```

pipe 方法是消费一个流的最简单方法，消费一个流一般有两种方法，一种就是用 pipe 方法，另外一种就是通过流的事件，我们在实际使用
过程中，尽量避免两者混合使用。另外 pipe 方法内部会自动帮我们处理很多东西比如：处理错误、文件结束符以及当一个流的流速要比另外一个
流的流速要快等等。

### 实现流

上面我们说的都是如何去消费一些特定的内置流，但有时候我们也要自定义流，下面我们说说实现流。

#### 实现一个可写流

我们可以基于 stream 模块的 Writable 来实现一个可写流

```Javascript
    const {Writable}=require('stream');
```

然后通过继承 Writable 类

```Javascript
    class MyWritableStream extends Writable{

    }
```

但最简单的我们可以直接实例化 Writable 对象，并传入参数，必须的参数是 write 方法，用于实现写入一块(chunk)数据

```Javascript
    const {Writable}=require('stream');
    const outStream=new Writable({
        // 三个参数：chunk一般是Buffer,encoding是数据编码，callback是写数据后的回调，主要是通知调用方结果，如果有异常可以将异常传入
        write(chunk,encoding,callback){
            console.log(chunk.toString());
            callback();
        }
    })
    process.stdin.pipe(outStream);
```

上面其实就是实现了一个相当于 process.stdout 的可写流

#### 实现可读流

类似的，我们可以通过 Readable 来实现可读流,我们需要实现 read 方法

```Javascript
    const {Readable}=require('stream');
    const readableStr=new Readable({
        read(){

        }
    })
```

然后用实例的 push 方法，将可读数据丢进去：

```Javascript
    readableStr.push('ABCDEFGHIJKLM');
    readableStr.push('NOPQRSTUVWXYZ');
    readableStr.push(null); //告知流没有更多的数据了
    readableStr.pipe(process.stdout);
```

但上面是一股脑把数据都推入流中，等待消费者来消费数据，更高端的写法应该是按需推数据：

```Javascript
    const {Readable}=require('stream');
    const readStr=new Readable({
        read(size){
            this.push(String.fromCharCode(this.currentCharCode));
            if(this.currentCharCode > 90){
                this.push(null)
            }
        }
    })
    readStr.currentCharCode=65;
    readStr.pipe(process.stdout);
```

当有消费者来读取该可读流时，read 方法会一直被调用，这样我们就往流推入更多的数据，直到边界 push null 告知消费者没有更多数据了

#### 实现 Duplex 流

这是一个双工流，既能可读又可写，实际就是把上面两种流的方法实现了

```Javascript
    const {Duplex}=require('stream');
    const duplexStr=new Duplex({
        write(chunk,encoding,callback){
            console.log(chunk.toString());
            callback();
        },
        read(size){
            this.push(String.fromCharCode(this.currentCharCode++));
            if(this.currentCharCode > 90){
                this.push(null);
            }
        }
    })

    process.stdin.pipe(duplexStr).pipe(process.stdout);
```

#### 实现 Transform 流

这个装换流并不用像双工流那样实现 read、write 方法，只要实现 transform 方法就好

```Javascript
    const {Transform}=require('stream');
```

#### 流对象模式

默认情况下，流接受 Buffer 或字符串类型的数据，不过有一个 objectMode 参数，我们可以通过设置它来使得流接受其他 Javascript 对象

```Javascript
    const {Transform}=require('stream');
    const commaSplitter=new Transform({
        readableObjectMode:true,  // 可将对象推入流中
        transform(chunk,encoding,callback){
            this.push(chunk.toString().trim().split(',')); // 将数据推入流
            callback();
        }
    })
    const arrayToObject=new Transform({
        readableObjectMode:true, //可将对象推入可读流
        writableObjectMode:true, //可接收对象的可写流
        transform(chunk,encoding,callback){
            const obj={};
            for(let i=0,len=chunk.length;i<len;i++){
                obj[chunk[i]]=chunk[i+1];
            }
            this.push(obj);
            callback();
        }
    })
    const objectToString=new Transform({
        writableObjectMode:true,
        transform(chunk,encoding,callback){
            this.push(JSON.stringify(chunk)+"\n");
            callback();
        }
    })
    process.stdin.pipe(commaSplitter)  //输入 a,b,c,d
    .pipe(arrayToObject)
    .pipe(objectToString)
    .pipe(process.stdout);
```

### 文件操作

文件操作是一门语言很重要的一项操作，Node 为我们提供了一个 fs 的核心模块，提供了大量的 API 来
做一些涉及到文件的操作的。

### 多进程

Node 运行在单进程单线程上，非阻塞 I/O 超级强大，这给用户带来不少好处的同时也存在一些问题：

- 1、如何利用好机器的多核 CPU？
- 2、如果是 CPU 密集型任务，

### 服务器开发
