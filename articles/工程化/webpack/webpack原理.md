### 查找webpack入口文件

webpack最终会查找到webpack-cli或者是webpack-command这个npm包，然后执行CLI

### webpack-cli做的事情

- 引入yargs，对命令行进行定制
- 分析命令行参数，对各个参数进行转换，组成编译配置项
- 引入webpack，根据配置项进行编译和构建


### webpack的本质

可以理解webpack是一种基于事件流的编程范例，一系列插件的运行。