### 闭包 和 作用域

### 作用域

作用域是指在程序中定义变量的区域，作用域规定了变量与函数的活动范围。

在es6之前，js中只有全局作用域和函数作用域。在这之后，引入了let 和 const ，它们会创建块级作用域。

在函数调用栈中，当内部函数查找一个外部变量，则js引擎就会向上级作用域中查找。这个查找的链条称为作用域链。

#### 闭包

闭包的目的是能在函数的外部作用域中访问函数内部的上下文。通过作用域以及作用域链，我们知道了当前作用域中（outer）的上下文(变量)，会在当前调用栈结束调用后被gc销毁。但是通过作用域链，我们可以在一个内部函数（inner）中访问outer作用域中的变量，创建引用关系。这样，就算调用栈执行完毕后也不会销毁outer中被引用的执行上下文。

```javascript

function outer() {
    // outer
    var a = 1;
    return {
        getA: function() {
            // inner
            return a;
        },
        setA: function(v) {
            // inner
            a = v;
        }
    }
}

```


### 你是如何组织自己的代码？是使用模块模式，还是使用经典继承的方法？

正常些js方法时使用模块模式，一个IIFE就算是一个模块，互相之间不影响。

继承在react跟node中也是比较常用的。

```javascript
var jspy = (function() {
  var _count = 0;

  var incrementCount = function() {
    _count++;
  }

  var getCount = function() {
    return _count;
  }

  return {
    incrementCount: incrementCount,
    getCount: getCount
  };

})();  
```




### 匿名函数的典型用法

匿名函数就是没有名称的函数, 如：
```javascript
(function () {})()

!function (){}()

void function () {}() 

function foo() {
    return function() {}
}
```

### [宿主对象(host object) 和原生对象(native object)、内置对象的区别](https://blog.csdn.net/weixin_40387601/article/details/80431670)

### 请指出以下代码的区别：function Person(){}、var person = Person()、var person = new Person()？

1. 定义了一个名为Person的函数
2. 普通的调用Person函数，并将返回值赋值到person变量上。
3. 使用new关键字，将Person作为构造函数实例化


### call apply bind

[call, apply实现](https://github.com/mqyqingfeng/Blog/issues/11)
[bind实现](https://github.com/mqyqingfeng/Blog/issues/12)

```javascript
Function.prototype.call = function(context) {
    // this传参为null则指向window
    var context = context || window;
    
    context.fn = this;

    var args = [];

    for (var i = 1, len = arguments.length; i < len ; i++) {
        args.push('arguments[' + i + ']');
    }
    
    var result = eval('context.fn(' + args +')');
    delete context.fn;
    return result;
}

Function.prototype.apply = function(context, arr) {
    var context = Object(context) || window;
    
    context.fn = this;

    var result;

    if (!arr) {
        result = context.fn();
    } else {
        var args = [];
        for (var i = 0 , len = arr.length; i< len ; i++) {
            arr.push('arr['+i+']');
        }
        result = eval('context.fn(+' args '+)')
    }
    delete context.fn
    return result;
}

Function.prototype.bind = function(context) {
    var outerArgs = Array.prototype.slice.call(arguments, 1);
    // 实例
    var self = this;

    return function() {
        var innerArgs = Array.prototype.slice.call(arguments);
        var args = outerArgs.concat(innerArgs)
        return self.apply(context, args)
    }
}
```

### [new 实现](https://github.com/mqyqingfeng/Blog/issues/13)

```javascript
function fakeNew() {
    var obj = Object();
    var Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;
    var ret = Constructor.apply(obj, arguments);

    return typeof ret === 'object' ? ret :obj;
}
```

### 函数防抖和函数节流
**防抖函数概念：** 
> 该函数只能在指定延时结束后才能调用，如果在过程中重复调用，则重新计时。

**应用场景:** 
1. 用户在搜索框输入时的数据查询，指定`n` 毫秒 延时，只能在输入完后过了`n`毫秒后才会去搜索，中间的持续输入则会重新计时
2. 在监听window.onresize事件，并触发某些操作时。
```js

function debounce(fn, interval) {
  var timer;
  return function() {
    var _self = this;
    if (timer) clearTimeout(timer);

    timer = setTimeout(function() {
      fn.apply(_self, arguments)
    }, interval)
  }
}

```

**函数节流:** 
> 在规定时间内，函数只能被调用一次。如果单位时间内多次触发，则忽略

**应用场景：**
1. 点击搜索按钮时的防重复。
2. 监听scroll事件时

```js
function throttle(fn, interval) {
  var last = 0;
  return function() {
    var now = +new Date();
    if (now - last >= interval) {
      fn.apply(this, arguments)
      last = +new Date();
    }
  }

}
```


### 什么情况下会使用document.write()

- 加载需要配合js脚本使用的外部css文件
```javascript
<scirpt>
document.write('<link  rel="stylesheet" href="style_neads_js.css">');
</script>
```

- 在新打开的页面中写入数据时
由于document.write会重写整个页面，异步调用会影响本页面的文档，如果在新窗口空白页调用，就没影响了。新开一个窗口，把本页面取到的数据在新窗口展示。

```javascript
document.open();
document.write('hello world');
document.close();
```

### [特性检测、特性腿短、浏览器UA字符串嗅探](https://blog.csdn.net/qq402164452/article/details/54882332)

### [Ajax工作原理](https://cloud.tencent.com/developer/article/1129601)

### [跨域方案](https://juejin.im/post/5c23993de51d457b8c1f4ee1)

[图片Ping](https://www.cnblogs.com/xiaohuochai/p/6567712.html)
图片Ping是客户端向服务器的单向通信，因为src请求资源不属于同源策略，所以一般可以用来做埋点、前端监控，比如监听网页的PV(Page View)，UV(Unique Visitor)。
最好是采用1 * 1 像素的透明gif图，因为：
1. 能够完成整个 HTTP 请求+响应（尽管不需要响应内容）
2. 触发 GET 请求之后不需要获取和处理数据、服务器也不需要发送数据
3. 跨域友好
4. 执行过程无阻塞
5. 相比 XMLHttpRequest 对象发送 GET 请求，性能上更好
6. GIF的最低合法体积最小（最小的BMP文件需要74个字节，PNG需要67个字节，而合法的GIF，只需要43个字节）

### [js埋点方案](http://unclechen.github.io/2017/12/24/JS%E5%9F%8B%E7%82%B9%E6%8A%80%E6%9C%AF%E5%88%86%E6%9E%90/)
1. 手动埋点
    - 也就是代码埋点，通过引入SDK，主动调用方法进行数据上报。代码侵入大，在时间充分的情况下是首选
2. 无痕埋点
    - 通过实现JS SDK。对整个文档的可点击元素如: `a、img、button`等进行事件监听，并上报。但是这个方法非常的暴力，会产生大量的无用数据。建议一些按钮不多，页面相对简单的页面使用

3. 可视化埋点（可视化也是上报全量点击数据，只是在后台根据可视化配置过滤出实际的数据）
    - 标记元素，保存配置：这一步要保存好需要追踪的元素的element_path，以及需要追踪的元素。
    - 下发配置，查找元素，监听点击，上报行为：这一步要通过element_path找到元素，给它添加一个点击监听器，当点击事件发生时SDK上报事件。

| 埋点方案	| 优点	| 缺点	| 适用场景
| - | - | - | - | 
代码埋点 | 	可以按照业务上报详细、定制化的数据	| 需要开发人员参与，更新维护成本高，无法获得历史数据	|  对上下文理解要求较高的业务数据
全埋点	| 对发人员依赖低，仅需嵌入一次SDK，可以全量上报通用数据，可以拿到历史数据	| 数量量太大，占用更多资源，且无法收集业务上下文数据，给后续数据筛选和分析带来一定的难度 | 	上下文相对独立的、通用的数据 | 
可视化埋点	| 对开发人员依赖低，可以按照业务需求上报数据，对上下文数据有一定收集能力 | 	标记事件有一定的操作难度，事件需要被更新时无法获得历史数据，界面变化时标记的元素可能失效 | 	业务上下文数据相对简单，操作交互比较固定的界面

### [变量声明提升](https://github.com/lxnxbnq/blog/issues/12)

### [js冒泡机制](https://blog.csdn.net/luanlouis/article/details/23927347)

### property(属性)和attribute(特性)

- DOM有其默认的基本属性，而这些属性就是所谓的“property”，无论如何，它们都会在初始化的时候再DOM对象上创建。

- HTML标签中定义的属性和值会保存该DOM对象的attributes属性里面；
- 这些attribute属性的JavaScript中的类型是Attr，而不仅仅是保存属性名和值这么简单；


```html
<input id="in_2" sth="whatever">
```
```javascript
var in2 = document.getElementById('in_2');
console.log(in2);
// id: "in_2"
// value: null


console.log(in1.attibutes.sth);     // 'sth="whatever"'
```

### document load 和 document DOMContentLoaded

- load方法在网页中所有的资源（HTML,CSS,image）都完全加载后才触发
- 当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完成加载

### [== 和 === 有什么不同](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness)

### [同源策略 (same-origin policy)](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

### [strict模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)

### 为何通常会认为保留网站现有的全局作用域 (global scope) 不去改变它，是较好的选择

- 减少命名冲突
- 有利于模块化

### 请解释什么是单页应用 (single page app), 以及如何使其对搜索引擎友好 (SEO-friendly)

单页应用是指在浏览器中运行的应用，它们在使用期间不会重新加载页面。像所有的应用一样，它旨在帮助用户完成任务，比如“编写文档”或者“管理Web服务器”。可以认为单页应用是一种从Web服务器加载的富客户端

使用[服务端渲染](http://www.alloyteam.com/2015/10/8783/)

### [Promise如何使用，实现](https://github.com/xieranmaya/blog/issues/3)

### [可变对象和不可变对象](https://blog.csdn.net/hustzw07/article/details/77946698)

#### 可变对象

我们知道，JavaScript中对象是弱类型的。一般情况下，可以不受限制的为对象添加属性，修改属性，删除属性。大部分情况下，我们使用的都是可变对象。

#### 不可变对象

对应的，我们不希望代码中某些对象被任意修改，比如添加、修改、删除等。这就是我们的不可变对象。JavaScript为我们提供了一些原生方法，借助它们可以讲一些可变对象转变成不可变对象。一共有三种：不可扩展，密封，冻结。

1. **Object.preventExtensions(obj)** 不可扩展（无法阻止深层属性的扩展）

```javascript

function Person(name) {
	this.name = name;
}

var person1 = new Person('william');
Object.preventExtensions(person1); // 阻止扩展属性
person1.age = 10; 

console.log(person1) // {name: 'william'}

var person2 = {
    name: 'william',
    info: {
        age: 23,
        height: '183cm'
    }
}

Object.preventExtensions(person2);
person2.info.sex = 'male'; // 可以扩展深层属性

person2.info.sex; // male 

```

2. **Object.seal(obj)** 方法可以让一个对象密封，并返回被密封后的对象。密封对象将会阻止向对象添加新的属性。另外也会 改变属性的 configurable 描述。


3. **Object.freeze(obj)** 方法可以冻结一个对象。冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。

### [什么是事件循环](https://zhuanlan.zhihu.com/p/33058983)

### [let var const](https://github.com/lxnxbnq/blog/issues/12)

### [数组的方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

### [web worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)

### [函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42)

### 创建对象的三种方法

- 构造函数: `var obj = new Object();`
- 对象字面量: `var obj = {};`
- `Object.create(obj[, otherobj])`;

### [深拷贝和浅拷贝](https://juejin.im/post/59ac1c4ef265da248e75892b)

```javascript
var obj = {
  name: 'william',
  age: 20,
  other: {
    work: 'code',
    eat: ['胡萝卜', '西瓜', {test: 1}],
  },
}

var arr = [1,2,3, [4,5, {test: 6}]]

function deepClone(data) {
  var type = Object.prototype.toString.call(data);
  var newData;

  if (type === "[object Object]") {
    newData = {};
    for (var key in data) {
      if (
        Object.prototype.toString.call(data[key]) !== "[object Object]" &&
        Object.prototype.toString.call(data[key]) !== "[object Array]" &&
        hasOwnProperty.call(data, key) &&
        !hasOwnProperty.call(newData, key)
      ) {
        newData[key] = data[key]
      } else {
        newData[key] = deepClone(data[key])
      }
    }
  } else if (type === "[object Array]") {
    newData = [];
    for (var i =0; i < data.length; i++) {
      if (
        Object.prototype.toString.call(data[i]) !== "[object Object]" &&
        Object.prototype.toString.call(data[i]) !== "[object Array]"
      ) {
        newData.push(data[i])
      } else {
        newData.push(deepClone(data[i]))
      }
    }
  } else {
    return data;
  }
  return newData;
}
```

### [网页上各种高度](https://blog.csdn.net/gs6511/article/details/53900761)

### [实现页面加载进度条](https://github.com/rstacruz/nprogress/)

### 箭头函数和普通函数有什么区别

[箭头函数](https://zhuanlan.zhihu.com/p/59376805)

||箭头函数|普通函数|
|-|-|-|
|写法 | var fn = () => {}| function fn() {} |
|arguments| 无 | 有|
|this | 运行时向父作用域查找，直到window | 1. 作为函数调用，指向window 2. 作为对象的方法调用，指向当前对象|
|new | 不能使用 | 函数作为构造函数 |
|原型属性 | 没有原型属性`console.log(fn.prototype) //undefined` | 有原型属性
