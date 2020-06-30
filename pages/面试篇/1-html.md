### **DOCTYPE(文档类型)**的作用是什么?

DTD(document type definition, 文档类型定义) 是一系列的语法规则，用来定义XML或者(X)HTML的文件类型。浏览器通过它来判断文档类型,并执行文件合法性的检验，并决定使用何种协议来解析，以及切换浏览器模式(标准模式或者混杂模式)

### [文档的混杂模式(quirks mode)和标准模式(standards mode)](https://github.com/lxnxbnq/blog/issues/16)

### 使用 data- 属性的好处是什么？

可以存储一些不需要显示在浏览器上的额外信息，比如一片文章，并且你想要的存储一些额外的信息。
```html
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

### 如果将HTML5看做一个开放平台，那它的构建模块有哪些？

标签及属性、地理位置(Geolocation)、画布(canvas)、视频(video)、音频(audio)、拖放(drag)、[微数据](https://www.zhangxinxu.com/wordpress/2011/12/html5%E6%89%A9%E5%B1%95-%E5%BE%AE%E6%95%B0%E6%8D%AE-%E4%B8%B0%E5%AF%8C%E7%BD%91%E9%A1%B5%E6%91%98%E8%A6%81/#comments)、[应用缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache)、Web存储、web workers、[服务器发送事件](https://developer.mozilla.org/zh-CN/docs/Server-sent_events/Using_server-sent_events)

### cookie、sessionStorage、localStorage、indexedDB的区别

|           |    cookie   |    sessionStorage   |   localStorage    |
| ----      |     -----   |    -----            |    -----          |
|   请求方式    |   cookie始终在同源的http请求中被携带    |   请求中不主动携带   |  请求中不主动携带 |
|   数据大小    |   4K(4 * 1024 Byte)    |    5M(5 * 1024 * 1024)   |  5M |
|   数据有效期    |    只在设置cookie过期时间之前有效   |   仅在当前浏览器窗口关闭前有效   |  永久存储 |
|   作用域    |    所有同源窗口中共享   |   所有同源窗口中共享   |  不在不同的浏览器窗口中共享，即使是同一个页面 |

[indexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/indexedDB)

### `<script>`、`<script defer>` 、`<script async >` 的区别

![](../../assets/script标签区别.png)

通过上图可知，在解析HTML时遇上script标签
1. 当为 **`<script/>`** 时，渲染进程会停止html解析并下载script标签的content然后执行，在这个过程中阻塞解析
2. 当为 **`<script defer/>`** 时，渲染进程不会停止html解析，而是并行下载script，然后等到html解析完成后，**DOMContentLoaded** 事件触发之前执行
3. 当为 **`<script async/>`** 时，并行下载script，然后执行script脚本，并阻塞html解析

### 为什么通常推荐将 CSS `<link>` 放置在 `<head></head>` 之间，而将 JS `<script>` 放置在 `</body>` 之前？你知道有哪些例外吗？
把 **`<link>`** 标签放在 **`<head></head>`** 之间是规范要求的内容。
此外，这种做法可以让页面逐步呈现，提高了用户体验。
将样式表放在文档底部附近，会使许多浏览器（包括IE）不能逐步呈现页面。
一些浏览器会阻止渲染，以避免在页面样式发生变化时，重新绘制页面中的元素。这种做法可以防止呈现给用户空白的页面或没有样式的内容。

把 **`<script>`** 标签恰好放在 **`</body>`** 之前

脚本在下载和执行期间会阻止HTML解析。把 **`<script>`** 标签放在底部，保证HTML首先完成解析，将页面尽早呈现给用户。

例外情况是当你的脚本里包含`document.write()`时。但是现在，`document.write()`不推荐使用。
同时，将 **`<script>`** 标签放在底部，意味着浏览器不能开始下载脚本，直到整个文档（document）被解析。
比较好的做法是，`<script>` 使用 **defer** 属性，放在 **`<head>`** 中


### 什么是渐进式渲染（progressive rendering）？
渐进式渲染是用于提高网页性能（尤其是提高用户感知的加载速度），以尽快呈现页面的技术。

在以前互联网带宽较小的时期，这种技术更为普遍。如今，移动终端的盛行，而移动网络往往不稳定，渐进式渲染在现代前端开发中仍然有用武之地。

例如:

1. 图片懒加载: 页面上的图片不会一次性全部加载。当用户滚动页面到图片部分时，JavaScript将加载并显示图像。
2. 确定显示内容的优先级（分层次渲染）: 为了尽快将页面呈现给用户，页面只包含基本的最少量的CSS、脚本和内容，然后可以使用延迟加载脚本或监听DOMContentLoaded/load事件加载其他资源和内容。
3. 异步加载HTML片段——当页面通过后台渲染时，把HTML拆分，通过异步请求，分块发送给浏览器。更多相关细节可以在[这里](https://tech.ebayinc.com/engineering/async-fragments-rediscovering-progressive-html-rendering-with-marko/)找到。


### HTML 和 XHTML 有什么区别？
XHTML是当前HTML版的继承者。HTML语法要求比较松散，这样对网页编写者来说，比较方便，但对于机器来说，语言的语法越松散，处理起来就越困难，对于传统的计算机来说，还有能力兼容松散语法，但对于许多其他设备，比如手机，难度就比较大。因此产生了语法要求更加严格的XHTML。

### [HMTL5新标签](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5/HTML5_element_list)

### iframe的使用
1. https://segmentfault.com/a/1190000004502619
2. https://www.zhihu.com/question/20653055