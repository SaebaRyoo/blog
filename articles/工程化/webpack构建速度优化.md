### speed-measure-webpack-plugin
使用该插件分析打包速度

### webpack-bundle-analyzer
分析打包体积

### 使用高版本的webpack和node

### 多进程/多实例构建

### 多进程并行代码压缩

### 预编译资源模块

### 利用缓存提升二次构建速度

思路：
- babel-loader 开启缓存
- terser-webpack-plugin开启缓存
- 使用cache-loader或者hard-source-webpack-plugin 针对模块的缓存开启