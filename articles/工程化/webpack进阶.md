### 多页面应用（MPA）
每个页面对应一个entry，一个html-webpack-plugin

缺点： 每次新增或删除页面需要改webpack配置

解决方案：

动态获取entry和设置html-webpack-plugin数量。

利用glob.sync,以同步的方式将页面查询出来
```js

module.exports = {
    entry: glob.sync(path.join(__dirname, './src/*/index.js'))
}
```

将对应的资源放在对应的文件夹中


### tree shaking（摇树优化）

概念: 一个模块可能有很多方法，只要其中某个方法使用到了，则整个文件都打包到bundle中。tree shaking就是只把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除掉。

使用: webpack默认支持，在babelrc里设置modules: true即可，production mode的情况默认开启

要求: 必须使用es6的语法。cjs的方式不支持

**原理**

DCE（Elimination）

代码不会被执行到，不可到达
```js
if (false) {
    // todo
}
```

代码执行的结果不会被用到,比如下面这段函数，如果他返回的结果没有被使用到。那么这段代码会被优化
```js
function f(a, b) {
    return a*b
}
```

代码只会改变死变量（只写不读），比如下面的变量a，没有被任何代码使用到。那么function和这个变量会被优化
```js
let a;

function f() {
    a = 20;
}
```

利用es6模块的特点：
- 只能作为模块顶层的语句出现
- import 的模块只能是字符串敞亮
- import binding 是immutable的

tree shaking的特点就是 静态的分析代码中哪些模块被用到了，不能像cjs那种，在运行阶段可以动态引入模块。
tree shaking需要明确知道哪些模块被用到，这样在编译阶段才能区分哪些代码可以被去除。
他会将没有用到的代码增加一些标识。然后在uglify阶段删除无用的代码。且代码中不能有副作用(网络请求、DOM操作等)。这一类的会导致tree shaking失效


### 代码分割和动态import；
