# WebWorker介绍

### 什么是Web Worker？

为JS创造多线程的环境，允许主线程创建Worker线程，将一些任务分配给后者运行。也就是主线程和worker线程并行。等到Worker线程完成计算的任务，再通过postMessage与主线程进行通信。

### 为什么需要Web Worker？
随着电脑计算能力的加强。js原有的单线程模型，无法很好的发挥这些计算能力。而web worker就是为了发挥更好的计算能力。这样，他能将一些计算密集或者高延时的任务，放在Worker线程中执行。主线程负责UI的交互，这样不会被阻塞。


### 特点
1. 同源限制： 分配给Worker线程运行的脚本文件，需要和主线程的脚本同源
2. DOM限制： Worker 线程所在的环境的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以navigator对象和location对象。
3. 通信： Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。
4. 脚本限制： worker线程不能执行 alert、confirm方法，但是可以使用XMLHttpRequest对象发出ajax请求
5. 文件限制： 无法读取本地文件

### 简单应用
在写之前，我们需要确保worker线程的脚本文件和主线程的脚本文件是同源的。所以我们可以通过node来作为web server。日常开发中可以使用webpack的[worker-loader](https://www.webpackjs.com/loaders/worker-loader/)

index.js
```js
var express = require('express')
var ejs = require('ejs')
var app = express()

const options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now())
  }
}

app.use(express.static('statics', options))

app.set('views','templates');
app.set('view engine','ejs');
app.engine('ejs', ejs.__express);       //定义模板引擎

app.get('/worker', function(req,res) {
  res.render('worker.ejs');
})

app.listen(3000, () => {
  console.log('服务已开启')
})
```

然后，我们在同级目录中添加`templates`和`statics`目录。

然后再templates目录中增加 `worker.ejs`
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Web worker</title>
    </head>
    <body>
    </body>

    <script>
        var worker = new Worker('/worker.js')


        // 向worker发送信息
        worker.postMessage('Hello World');

        // 监听worker发送的信息
        worker.onmessage = function (e) {
            console.log('received message from service ' + e.data);
        }

    </script>
</html>
```

然后再在statics目录中添加`worker.js`
```js

// 监听 message
// self 代表子线程，即子线程的全局对象，等同于this
self.addEventListener('message', function (e) {
    console.log('received message from chrome', e.data)
    this.setTimeout(function() {
        self.postMessage('hihi');
    }, 1000)
}, false);
```
运行`node index.js`，然后在浏览器中访问`http://localhost/worker`，你就会发现console面板中会输出如下:

1. received message from chrome Hello World
2. 间隔1秒左右，再输出 received message from service hihi



# Service Worker
### 什么是service worker呢？

service worker相当于一个在浏览器和服务器之间的中间人。如果网站注册了service worker，那么它可以拦截当前网站的所有请求，进行判断。如果需要向服务器发起请求的就转给服务器，如果可以直接使用缓存的就直接返回缓存不再转给服务器。从而大大提高浏览体验
