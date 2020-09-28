### css 中的 class和id有什么区别？
1. 权重不同
2. class是可重复的； id是唯一的，如果有多个相同的id，只去文档中的最后一个
3. id可以用来在页面中做锚点，当地址栏的hash和页面中某个id值一致，页面会自动滚动到该元素上

### 请问 "resetting" 和 "normalizing" CSS 之间的区别？你会如何选择，为什么？

1. resetting

CSS Reset让各个浏览器的CSS样式有一个统一的基准，而实现这一基准最主要的方式就是“清零”

2. normalizing

Normalize.css 只是一个很小的CSS文件，但它在默认的HTML元素样式上 提供了跨浏览器的高度一致性。相比于传统的CSS reset，Normalize.css 是一种现代的、为HTML5准备的优质替代方案。


### 请解释css浮动(Float)以及其工作原理
- 浮动元素从网页的正常流中移出，但是保留了部分流动性。会影响其他元素的定位(比如文字会围绕着浮动元素)。这一点和绝对定位(absolute)不同，absolute元素完全脱离的文档流
- 如果浮动元素的父元素只包含浮动元素，那么该父元素的高度会坍塌为0，我们可以通过清除从浮动元素后到父元素关闭前之间的浮动来修复这个问题(`clear: both | left | right`)。如， 给父元素加伪类
```css
  .father::after {
      content: ' ';
      display: block;
      clear: both;
  }
```
- 把浮动元素的父元素属性设置为`overflow: auto | hidden`,会使其内部形成 **BFC** , 并且父元素会扩张自己，使其能够包围它的子元素

### 请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局

css盒模型本质上是一个盒子，封装周围的HTML元素

#### 组成部分
Margin(外边距) - 清除边框外的区域，外边距是透明的。
Border(边框) - 围绕在内边距和内容外的边框。
Padding(内边距) - 清除内容周围的区域，内边距是透明的。
Content(内容) - 盒子的内容，显示文本和图像。

可以使用css3的[box-sizing](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing)属性来定义不同的和模型

```css
/** 
这是我们熟悉的符合W3C标准的盒子模型
尺寸计算公式：
width = 内容的宽度
height = 内容的高度
 **/
box-sizing: content-box 

/** 
 此时的盒子模型符合IE6以前的版本，知识IE的bug,盒子的宽度包括了border和padding
  尺寸计算公式：
  width = border + padding + 内容的宽度
  height = border + padding + 内容的高度
 **/
box-sizing: border-box 
```

### [什么是IFC CSS2.1](https://juejin.im/entry/5938daf7a0bb9f006b2295db)
IFC(Inline Formatting Contexts)直译为"内联格式化上下文"，IFC的line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响)
IFC中的line box一般左右都贴紧整个IFC，但是会因为float元素而扰乱。float元素会位于IFC与与line box之间，使得line box宽度缩短。 同个ifc下的多个line box高度会不同。 IFC中时不可能有块级元素的，当插入块级元素时（如p中插入div）会产生两个匿名块与div分隔开，即产生两个IFC，每个IFC对外表现为块级元素，与div垂直排列。
那么IFC一般有什么用呢？

水平居中：当一个块要在环境中水平居中时，设置其为inline-block则会在外层产生IFC，通过text-align则可以使其水平居中。

垂直居中：创建一个IFC，用其中一个元素撑开父元素的高度，然后设置其vertical-align:middle，其他行内元素则可以在此父元素下垂直居中。

### 请描述 BFC(Block Formatting Context) 及其如何工作？ CSS2.1
BFC是一种属性，它会影响元素的定位以及与其兄弟元素之间的互相作用。 中文常译为 **块级格式化上下文** 。是 W3C CSS 2.1 规范中的一个概念，它决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。 在进行盒子元素布局的时候，BFC提供了一个环境，在这个环境中按照一定规则进行布局不会影响到其它环境中的布局。比如浮动元素会形成BFC，浮动元素内部子元素的主要受该浮动元素影响，两个浮动元素之间是互不影响的。 也就是说，如果一个元素符合了成为BFC的条件，该元素内部元素的布局和定位就和外部元素互不影响(除非内部的盒子建立了新的 BFC)，是一个隔离了的独立容器。（在 CSS3 中，BFC 叫做 Flow Root）


#### 形成条件
1. 浮动元素，float 除 none 以外的值；
2. 绝对定位元素，position（absolute，fixed）；
3. display 为以下其中之一的值 inline-blocks，table-cells，table-captions；
4. overflow 除了 visible 以外的值（hidden，auto，scroll）

参考:
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

### 什么是GFC CSS3
GFC(GridLayout Formatting Contexts)直译为"网格布局格式化上下文"，当为一个元素设置display值为grid的时候，此元素将会获得一个独立的渲染区域，我们可以通过在网格容器（grid container）上定义网格定义行（grid definition rows）和网格定义列（grid definition columns）属性各在网格项目（grid item）上定义网格行（grid row）和网格列（grid columns）为每一个网格项目（grid item）定义位置和空间。
那么GFC有什么用呢，和table又有什么区别呢？首先同样是一个二维的表格，但GridLayout会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制。

### 什么是FFC CSS3
FFC(Flex Formatting Contexts)直译为"自适应格式化上下文"，display值为flex或者inline-flex的元素将会生成自适应容器（flex container），可惜这个牛逼的属性只有谷歌和火狐支持，不过在移动端也足够了，至少safari和chrome还是OK的，毕竟这俩在移动端才是王道。
Flex Box 由伸缩容器和伸缩项目组成。通过设置元素的 display 属性为 flex 或 inline-flex 可以得到一个伸缩容器。设置为 flex 的容器被渲染为一个块级元素，而设置为 inline-flex 的容器则渲染为一个行内元素。
伸缩容器中的每一个子元素都是一个伸缩项目。伸缩项目可以是任意数量的。伸缩容器外和伸缩项目内的一切元素都不受影响。简单地说，Flexbox 定义了伸缩容器内伸缩项目该如何布局。


### CSS sprites

优点：
1. 减少http请求
2. 合并图的字节比多张图要小

缺点：
1. 高分辨率的宽屏下，如果图片不够宽，很容易出现背景断裂
2. 开发过程繁琐，需要通过ps或其他测量工具精确的确认每一个背景单元的位置。
3. 维护复杂，当其中某个背景需要更新时，且大小发生变化，则大多数背景单元的位置都会发生变化


### [图片替换文字方案](https://cloud.tencent.com/developer/article/1033104)

### 你会如何解决特定浏览器的样式问题？

1. reset.css或者normalize.css： 解决不同浏览器对于元素初始化样式不一致问题
2. html5shiv.js: 解决ie9以下浏览器对html5新增标签不识别问题
```
<!--[if lt IE 9]>
  <script type="text/javascript" src="https://cdn.bootcss.com/html5shiv/3.7.3/html5shiv.min.js"></script>
<![endif]-->
```
3. respond.js： 解决ie9以下浏览器不支持 **css3 Media Query** 的问题
```
<script src="https://cdn.bootcss.com/picturefill/3.0.3/picturefill.min.js"></script>
```
具体请看[浏览器兼容性问题解决方案](https://juejin.im/post/59a3f2fe6fb9a0249471cbb4)


### 如何为有功能限制的浏览器提供网页？

#### 兼容
- 渐进增强的思路就是提供一个可用的原型，后来再为高级浏览器提供优化。
- 优雅降级的思路是根据高级浏览器提供一个版本，然后有功能限制的浏览器只需要一个刚好能用的版本。

#### 相关技术
- Media Query
- CSS hack
- 条件判断 <!--[if !IE]><!--> 除IE外都可识别 <!--<![endif]-->

### 有哪些隐藏内容的方法？
```css
overflow:hidden;
opacity:0;
visibility:hidden;
display:none;
position:absolute;
clip(clip-path):rect()/inset()/polygon();
z-index:-1000;
transform:scaleY(0);
```
具体请看[css隐藏元素的八种方法](https://juejin.im/post/584b645a128fe10058a0d625)

### [栅格系统 (grid system)](https://learnku.com/articles/21215)

### 你用过媒体查询，或针对移动端的布局CSS 吗？
- 媒体查询可用于响应式网站样式，也可针对不同型号手机的尺寸写不同的样式
- [通过rem布局](https://blog.csdn.net/lvyang251314/article/details/82798858)
- 通过vw/vh布局移动端css

### [如何优化网页的打印样式？](https://www.html.cn/archives/4731)

### 在书写高效的css时有哪些问题需要考虑
1. 样式是：从右向左的解析一个选择器
2. ID最快，Universal最慢 有四种类型的key selector，解析速度由快到慢依次是：ID、class、tag和universal
3. 不要tag-qualify （永远不要这样做 ul#main-navigation { } ID已经是唯一的，不需要Tag来标识，这样做会让选择器变慢。）
4. 后代选择器最糟糕（换句话说，下面这个选择器是很低效的： html body ul li a { }）
5. 想清楚你为什么这样写
6. CSS3的效率问题（CSS3选择器（比如 :nth-child）能够漂亮的定位我们想要的元素，又能保证我们的CSS整洁易读。但是这些神奇的选择器会浪费很多的浏览器资源。）
7. 我们知道#ID速度是最快的，那么我们都用ID，是不是很快。但是我们不应该为了效率而牺牲可读性和可维护性

### 使用css预处理器的优缺点

**缺点：** 简单来说CSS预处理器语言较CSS玩法变得更高级了，但同时降低了自己对最终代码的控制力。更致命的是提高了门槛，首先是上手门槛，其次是维护门槛，再来是团队整体水平和规范的门槛。这也造成了初学学习成本的昂贵。

**优点：** 用一种专门的编程语言，为CSS增加了一些编程的特性，将CSS作为目标生成文件，然后开发者就只要使用这种语言进行编码工作。通俗的说，CSS预处理器用一种专门的编程语言，进行Web页面样式设计，然后再编译成正常的CSS文件，以供项目使用。CSS预处理器为CSS增加一些编程的特性，无需考虑浏览器的兼容性问题，例如你可以在CSS中使用变量、简单的逻辑程序、函数等等在编程语言中的一些基本特性，可以让你的CSS更加简洁、适应性更强、可读性更佳，更易于代码的维护等诸多好处。

### 如果设计中使用了非标准的字体，你该如何去实现？

1. 用图片代替
2. web fonts在线字库，如[Google Webfonts](https://fonts.google.com/)，[Typekit](http://www.chinaz.com/free/2012/0815/269267.shtml) 等等；

### [请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？](https://www.zhihu.com/question/24959507)

从后往前判断。

浏览器先产生一个元素集合，这个集合往往由最后一个部分的索引产生（如果没有索引就是所有元素的集合）。
然后向上匹配，如果不符合上一个部分，就把元素从集合中删除，直到真个选择器都匹配完，还在集合中的元素就匹配这个选择器了。

举个例子，有选择器：`body.ready #wrapper > .lol233`

先把所有 class 中有 lol233 的元素拿出来组成一个集合，然后上一层，对每一个集合中的元素，如果元素的 parent id 不为 #wrapper 则把元素从集合中删去。 再向上，从这个元素的父元素开始向上找，没有找到一个 tagName 为 body 且 class 中有 ready 的元素，就把原来的元素从集合中删去。

至此这个选择器匹配结束，所有还在集合中的元素满足。

大体就是这样，不过浏览器还会有一些奇怪的优化。

为什么从后往前匹配因为效率和文档流的解析方向。效率不必说，找元素的父亲和之前的兄弟比遍历所哟儿子快而且方便。关于文档流的解析方向，是因为现在的 CSS，一个元素只要确定了这个元素在文档流之前出现过的所有元素，就能确定他的匹配情况。应用在即使 html 没有载入完成，浏览器也能根据已经载入的这一部分信息完全确定出现过的元素的属性。为什么是用集合主要也还是效率。基于 CSS Rule 数量远远小于元素数量的假设和索引的运用，遍历每一条 CSS Rule 通过集合筛选，比遍历每一个元素再遍历每一条 Rule 匹配要快得多。

### [请描述伪元素 (pseudo-elements) 及其用途](https://www.jianshu.com/p/9086114e07d4)

### 请罗列出你所知道的 display 属性的全部值

display 的属性值有：none|inline|block|inline-block|list-item|run-in|table|inline-table|table-row-group|table-header-group|table-footer-group|table-row|table-column-group|table-column|table-cell|table-caption|inherit

其中常用的的有none、inline、block、inline-block。分别的意思是：

1、none： 元素不会显示，而且改元素现实的空间也不会保留。但有另外一个 visibility: hidden， 是保留元素的空间的。

2、inline： display的默认属性。将元素显示为内联元素，元素前后没有换行符。我们知道内联元素是无法设置宽高的，所以一旦将元素的display 属性设为 inline， 设置属性height和width是没有用的。此时影响它的高度一般是内部元素的高度（font-size）和padding。

3、block： 将元素将显示为块级元素，元素前后会带有换行符。设置为block后，元素可以设置width和height了。元素独占一行。

4、inline-block：行内块元素。这个属性值融合了inline 和 block 的特性，即是它既是内联元素，又可以设置width和height。

### 请解释 inline 和 inline-block 的区别

- inline(行内元素): 无法设置宽高,不独占一行, 竖直方向的margin(margin-top、margin-bottom)不生效。
- inline-block(行内块元素): 可以设置宽高，不独占一行

### 请解释 relative、fixed、absolute、static、sticky 元素的区别

|    | relative | fixed | absolute | static | sticky
| -- | -- | -- | -- | -- | -- |
| 含义 | 相对定位 | 固定定位 | 绝对定位 | 静态定位 | 相对定位
| 是否脱离正常文档流| 是 | 是 | 是| 否 | 否
|是否还占用正常文档流的位置 | 是 | 否 | 否 | - | 是
| 相对于什么定位| 元素本身在正常文档流的位置 | 相对于浏览器窗口 | 相对于static定位以外的第一个父元素左上角border与padding交界处定位 | - | 相对它的最近滚动祖先（当该祖先的overflow 是 hidden, scroll, auto, 或 overlay时） 和 最近块级祖先，包括table-related元素，基于top, right, bottom, 和 left的值进行偏移。

### 请问你有尝试过 CSS Flexbox 或者 Grid 标准规格吗

[Css Flexbox](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
[CSS Grid 布局完全指南](https://www.html.cn/archives/8510)


### [为什么响应式设计 (responsive design) 和自适应设计 (adaptive design) 不同？](https://www.qifeiye.com/%e4%b8%80%e5%bc%a0%e5%9b%be%e7%9c%8b%e6%87%82%e5%93%8d%e5%ba%94%e5%bc%8fresponsive-design%e5%92%8c%e8%87%aa%e9%80%82%e5%ba%94%e6%8a%80%e6%9c%afadaptive-design%e7%9a%84%e5%8c%ba%e5%88%ab/)

### [你有兼容 retina 屏幕的经历吗？如果有，在什么地方使用了何种技术？](https://github.com/Rashomon511/LearningRecord/issues/301)


### 请问为何要使用 translate() 而非 absolute position，或反之的理由？为什么？

通过absolute定位属性实现的移动，通过translate也可以实现，两者结合使用可以实现元素的居中。

**文档流上的差异：**

absolute会脱离文档流，而translate不会脱离文档流

**基点上的差异：**

absolute是基于第一个非static父元素的左上角border与padding交界处，而translate是子元素整体平移，没有所谓的基点而言，当然通过transform-origin改变旋转的基准点？

**视图属性上的差异：**

可以看出使用 translate 的例子的 offsetTop 和 offsetLeft 的数值与没有产生位移的元素没有然后区别，无论位移多少这两个数值都是固定不变的。

而使用相对定位的例子 offsetTop 和 offsetLeft 的数值则根据位移的长度发生了改变。

**动画表现上的差异：**

使用绝对定位的动画效果会受制于利用像素(px)为单位进行位移，而使用 translate 函数的动画则可以不受像素的影响，以更小的单位进行位移。

另外，绝对定位的动画效果完全使用 **CPU** 进行计算，而使用 translate 函数的动画则是利用 **GPU**，因此在视觉上使用 translate 函数的动画可以有比绝对定位动画有更高的帧数。

### 如果实现一个高性能的CSS动画效果？

[CSS动画的性能优化- 云+社区- 腾讯云](https://cloud.tencent.com/developer/article/1178691)
[实现达到60FPS 的高性能交互动画- 知乎](https://zhuanlan.zhihu.com/p/29729996)

### [css3动画](https://www.w3school.com.cn/css3/css3_animation.asp)

```css
@keyframes first {
  from { 
    background: yellow;
    width: 100px
  } to {
    background: green;
    width: 200px
  }
}

div {
  animation: first 2s; 
}
```

### [实现垂直居中和水平居中](https://segmentfault.com/a/1190000014116655)

### 小数