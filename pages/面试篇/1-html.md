### 1. **DOCTYPE(文档类型)**的作用是什么?

DTD(document type definition, 文档类型定义) 是一系列的语法规则，用来定义XML或者(X)HTML的文件类型。浏览器通过它来判断文档类型,并执行文件合法性的检验，并决定使用何种协议来解析，以及切换浏览器模式(标准模式或者混杂模式)

### 2. [文档的混杂模式(quirks mode)和标准模式(standards mode)](https://github.com/lxnxbnq/blog/issues/16)

### 3. 使用 data- 属性的好处是什么？

可以存储一些不需要显示在浏览器上的额外信息，比如一片文章，并且你想要的存储一些额外的信息。
```
<article
  id="electriccars"
  data-columns="3">
...
</article>
```
并且可以通过dataset来获取一个元素中的所有data-信息
```
var article = document.querySelector('#electriccars');
article.dataset.columns // 3
```

### 4. 如果将HTML5看做一个开放平台，那它的构建模块有哪些？

标签及属性、地理位置(Geolocation)、画布(canvas)、视频(video)、音频(audio)、拖放(drag)、[微数据](https://www.zhangxinxu.com/wordpress/2011/12/html5%E6%89%A9%E5%B1%95-%E5%BE%AE%E6%95%B0%E6%8D%AE-%E4%B8%B0%E5%AF%8C%E7%BD%91%E9%A1%B5%E6%91%98%E8%A6%81/#comments)、[应用缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache)、Web存储、web workers、[服务器发送事件](https://developer.mozilla.org/zh-CN/docs/Server-sent_events/Using_server-sent_events)

### 5. cookie、sessionStorage、localStorage、indexedDB的区别

|           |    cookie   |    sessionStorage   |   localStorage    |
| ----      |     -----   |    -----            |    -----          |
|   请求方式    |   cookie始终在同源的http请求中被携带    |   请求中不主动携带   |  请求中不主动携带 |
|   数据大小    |   4K(4 * 1024 Byte)    |    5M(5 * 1024 * 1024)   |  5M |
|   数据有效期    |    只在设置cookie过期时间之前有效   |   仅在当前浏览器窗口关闭前有效   |  永久存储 |
|   作用域    |    所有同源窗口中共享   |   所有同源窗口中共享   |  不在不同的浏览器窗口中共享，即使是同一个页面 |

[indexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/indexedDB)

### 6. <script> 、<script defer> 、<script async > 的区别

|      |  script  | <script defer> | <script async > |
| ----      |     -----   |    -----            |    -----          |
| 内联-放在dom元素之前  | 阻塞DOM解析和渲染 | 

