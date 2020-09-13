### loader的链式调用和顺序执行

loader 就是一个j函数，它的作用就是接收源码，并返回新的源码。

如果是多个loader串行，则执行顺序是从右向左
```js
module.exports = {
    rules: [
        {
            test: /\.less$/,
            use: [
                'style-loader',
                'css-loader',
                'less-loader'
            ]
        }
    ]
}
```

如上，编译less文件的执行顺序为 less-loader -> css-loader -> style-loader

那么，为什么是从后往前的呢？

首先，函数的组合有两种情况，
1. Unix中的pipline
2. Compose 

那么webpack中使用的就是Compose， 它的原理如下:
```js
compose = (f, g) => (...args) => f(g(...args))
```

我们可以通过如下的例子来验证
```js
// a-loader.js
module.exports = function(source) {
    console.log('loader a');
    return source
}

// b-loader.js
module.exports = function(source) {
    console.log('loader b');
    return source
}
```

然后，将loader引入到webpack的配置中

```js

module.exports = {
    entry: './src/index.js';
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    rules: [
        {
            test: /\.js$/,
            use: [
                path.resolve('./loaders/a-loader.js'),
                path.resolve('./loaders/b-loader.js'),
            ]
        }
    ]
}

```

### 插件的基本结构

```js
class MyPlugin{ // 1. 插件名称
    apply(compiler) {  // 2. 插件上的apply方法，注入compiler
        compiler.hooks.done.tap('My Plugin', () => { // hooks
            console.log('....') // 插件处理逻辑
        })
    }
}

module.exports = MyPlugin
```