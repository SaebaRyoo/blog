### speed-measure-webpack-plugin
使用该插件分析打包速度，针对信息定制优化方案

### webpack-bundle-analyzer
分析打包体积

### 使用高版本的webpack和node
webpack4 相对于 webpack3 构建时间降低了60%-98%

webpack4 的优化如下：
- v8 带来的优化 (for of 替代forEach、Map和Set代替Object、includes代替indexOf)
- 默认使用更快的md4 hash算法 替代 md5
- webpacks AST 可以直接从loader传递给AST，减少解析时间
- 使用字符串方法代替正则表达式

### 多进程/多实例构建
资源并行解析方案
- thread-loader （官方在webpack4中加入的，可以很好的替代HappyPack）
- parallel-webpack
- HappyPack （主要用在webpack3中）

### 多进程并行代码压缩
**1. 使用parallel-uglify-plugin**
```js
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');

module.exports = {
    plugins: [
        new ParallelUgilfyPlugin({
            uglifyJS: {
                output: {
                    beautify: false,
                    comments: false
                },
                compress: {
                    warnings: false,
                    drop_console: true,
                    collapse_vars: true,
                    reduce_vars: true
                }
            }
        })
    ]
}
```

**2.uglifyjs-webpack-plugin开启parallel 参数**
```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
    plugins: [
        new UglifyJsPlugin({
            uglifyOptions: {
                warnings: false,
                parse: {},
                compress: {},
                mangle: true,
                output: null,
                toplevel: false,
                nameCache: null,
                ie8: false,
                keep_fnames: false
            },
            parallel: true // 开启并行压缩
        })
    ]
}
```

**3.terser-webpack-plugin开启parallel 参数**

推荐使用这种webpack4中的方案
```js
const TerserPlugin = require('terser-webpack-plugin');
module.exports = {
    optimization: {
        minimizer: [
            new TerserPlugin({
                parallel: 4 // 默认为当前电脑cpu的两倍
            })
        ]
    }
}
```
### 预编译资源模块

分包： 设置Externals

思路： 将react、react-dom等类似的基础包通过cdn的方式引入，不打包到bundle文件中

方法： 使用html-webpack-externals-plugin

缺点： 项目中可能存在多个基础包，每个基础包都需要一个cdn地址，增加了请求数

```js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

module.exports = {
    plugins: [
        new HtmlWebpackExternalsPlugin({
            externals: [
                {
                    module: 'react',
                    entry: 'https://unpkg.com/browse/react@16.13.1/umd/react.production.min.js',
                    global: 'React',
                },
                {
                    module: 'react-dom',
                    entry: 'https://unpkg.com/browse/react@16.13.1/umd/react-dom.production.min.js',
                    global: 'ReactDOM',
                },
            ]
        })
    ]
}
```

**针对上面的方案，可以采用预编译资源模块。webpack官方的DLLPlugin进行分包**

思路： 将react、react-dom、redux、react-redux、react-router等基础包和业务基础包打包成一个文件

方法： DLLPlugin分包，DllReferencePlugin对manifest.json引用。引用manifest.json会自动关联DllPulugin中的包

一般我们需要单独创建一个`webpack.dll.js`文件
```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
    context: process.cwd(),
    resolve: {
        extensions: ['.js', '.jsx', '.json', '.less', '.css'],
        modules: [__dirname, 'node_modules']
    },
    entry: {
        // 制定需要分离的包
        libaray: [
            'react',
            'react-dom',
            'redux',
            'react-redux',
        ]
    },
    output: {
        filename: '[name].dll.js',
        path: path.resolve(__dirname, './build/library'),
        library: '[name]'
    },
    plugins: [
        new webpack.DllPlugin({
            name: '[name]',
            path: './build/library/[name].json'
        })
    ]
}

```

分好包之后，在实际的项目中。也就是webpack.config.js中引入
```js
module.exports = {
    plugins: [
        new webpack.DllReferencePlugin({
            // mainfest就是对我们要引入包的描述
            manifest: require('./build/library/manifest.json')
        })
    ]
}
```

### 利用缓存提升二次构建速度

思路：
- babel-loader 开启babel编译缓存
- terser-webpack-plugin开启压缩缓存
- 使用cache-loader或者hard-source-webpack-plugin 针对模块的缓存开启


### 缩小构建目标
目的：尽可能的少构建模块

比如 babel-loader 不解析 node_modules

优化resolve.modules 配置（减少模块搜索层级）

优化resolve.mainFields配置

优化resolve.extensions配置

合理使用alias


### 使用tree shaking擦除无用的js和css

如何删除无用的css文件？

PurifyCSS: 遍历代码，识别已经用到的css class

```js
const path = require('path');
const glob = require('glob');
const MinCssExtractPlugin = require('mini-css-extract-plugin');
const PurgecssPlugin = require('purgecss-webpack-plugin');

const PATH = {
    src: path.join(__dirname, 'src'),
}

module.exports = {
    module: {
        rules: [
            {
                test: /\.css/,
                use: [
                    MinCssExtractPlugin.loader,
                    'css-loader',
                ]
            }
        ]
    },
    plugins: [
        new MinCssExtractPlugin({
            filename: '[name].css'
        }),
        new PurgecssPlugin({
            paths: glob.sync(`${PATH.src}/**/*`, {
                nodir: true
            })
        })
    ]
}

```

uncss： HTML需要通过jsdom加载，所有的样式通过PostCSS解析，通过document.querySelector来识别在html文件里面不存在的选择器


### 使用webpack进行图片压缩


### 使用动态polyfill服务
目的： 只针对不支持某个功能的设备动态引入polyfill

Polyfill Service原理：每次打开网页，请求polyfill service，识别User Agent，下发不同的polyfill

