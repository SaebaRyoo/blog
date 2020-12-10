# [原文地址，本篇仅做记录学习](https://www.cnblogs.com/PeunZhang/p/5553574.html)


### npm install 安装模块

1. 基础语法
```shell
npm install (with no args, in package dir)
npm install [<@scope>/]<name>
npm install [<@scope>/]<name>@<tag>
npm install [<@scope>/]<name>@<version>
npm install [<@scope>/]<name>@<version range>
npm install <tarball file>
npm install <tarball url>
npm install <folder>

alias: npm i
common options: [-S|--save|-D|--save-dev|-O|--save-optional] [-E|--save-exact] [--dry-run]
```

2. 安装包，默认会安装最新的版本
`npm install gulp`

3. 安装指定版本
`npm install gulp@1.1.1`

- 项目对模块的依赖可以使用下面的 3 种方法来表示（假设当前版本号是 1.1.0 ）：

兼容模块新发布的补丁版本：~1.1.0、1.1.x、1.1
兼容模块新发布的小版本、补丁版本：^1.1.0、1.x、1
兼容模块新发布的大版本、小版本、补丁版本：*、x

- **-S, --save 安装包信息将加入到dependencies（生产阶段的依赖）**

`npm install gulp --save 或 npm install gulp -S`

package.json 文件的 dependencies 字段：

```json
"dependencies": {
    "gulp": "^3.9.1"
}
```

- **-D, --save-dev 安装包信息将加入到devDependencies（开发阶段的依赖），所以开发阶段一般使用它**

`npm install gulp --save-dev 或 npm install gulp -D`

package.json 文件的 devDependencies字段：
```json
"devDependencies": {
    "gulp": "^3.9.1"
}
```

- **-O, --save-optional 安装包信息将加入到optionalDependencies（可选阶段的依赖）**

`npm install gulp --save-optional 或 npm install gulp -O`

package.json 文件的optionalDependencies字段：

```json
"optionalDependencies": {
    "gulp": "^3.9.1"
}
```

- **-E, --save-exact 精确安装指定模块版本**

`npm install gulp --save-exact 或 npm install gulp -E`

输入命令**npm install gulp -ES**，留意package.json 文件的 dependencies 字段，以看出版本号中的^消失了

```json
"dependencies": {
    "gulp": "3.9.1"
}
```

模块的依赖都被写入了package.json文件后，他人打开项目的根目录（项目开源、内部团队合作），使用npm install命令可以根据dependencies配置安装所有的依赖包

- 本地安装 `npm install gulp`

- 全局安装 `npm install gulp -g`


### npm uninstall 卸载模块

1. 基础语法
```
npm uninstall [<@scope>/]<pkg>[@<version>]... [-S|--save|-D|--save-dev|-O|--save-optional]

aliases: remove, rm, r, un, unlink
```


### npm update 更新模块
    npm update [-g] [<pkg>...]

### npm outdated 检查模块是否已经过时
    npm outdated [[<@scope>/]<pkg> ...]

### npm ls 查看安装的模块
```
npm ls [[<@scope>/]<pkg> ...]

aliases: list, la, ll
```

### npm help 查看某条命令的详细帮助 
1. 基础语法
`npm help <term> [<terms..>]`

2. 例如输入npm help install，系统在默认的浏览器或者默认的编辑器中打开本地nodejs安装包的文件/nodejs/node_modules/npm/html/doc/cli/npm-install.html

 `npm help install`


### npm root 查看包的安装路径
    输出 node_modules的路径

    npm root [-g]

### npm config 管理npm的配置路径
    npm config set <key> <value> [-g|--global]
    npm config get <key>
    npm config delete <key>
    npm config list
    npm config edit
    npm get <key>
    npm set <key> <value> [-g|--global]


对于config这块用得最多应该是设置代理，解决npm安装一些模块失败的问题

例如我在公司内网，因为公司的防火墙原因，无法完成任何模块的安装，这个时候设置代理可以解决
    npm config set proxy=http://xxx.com:8080

又如国内的网络环境问题，某官方的IP可能被和谐了，幸好国内有好心人，搭建了镜像，此时我们简单设置镜像

    npm config set registry="http://r.cnpmjs.org"

也可以临时配置，如安装淘宝镜像

    npm install -g cnpm --registry=https://registry.npm.taobao.org

### npm cache 管理模块的缓存
    npm cache add <tarball file>
    npm cache add <folder>
    npm cache add <tarball url>
    npm cache add <name>@<version>

    npm cache ls [<path>]

    npm cache clean [<path>]


最常用命令无非清除npm本地缓存

    npm cache clean


### npm view 查看模块的注册信息
    npm view [<@scope>/]<name>[@<version>] [<field>[.<subfield>]...]

    aliases: info, show, v

查看模块的依赖关系

    npm view gulp dependencies
查看模块的源文件地址

    npm view gulp repository.url
查看模块的贡献者，包含邮箱地址

    npm view npm contributors


### npm adduser 用户登录

    npm adduser [--registry=url] [--scope=@orgname] [--always-auth]

### npm publish 发布模块
    npm publish [<tarball>|<folder>] [--tag <tag>] [--access <public|restricted>]

    Publishes '.' if no argument supplied
    Sets tag 'latest' if no --tag specified

### npm access 在发布的包上设置访问级别
    npm access public [<package>]
    npm access restricted [<package>]

    npm access grant <read-only|read-write> <scope:team> [<package>]
    npm access revoke <scope:team> [<package>]

    npm access ls-packages [<user>|<scope>|<scope:team>]
    npm access ls-collaborators [<package> [<user>]]
    npm access edit [<package>]


### npm package.json的语法

https://github.com/ericdum/mujiang.info/issues/6/