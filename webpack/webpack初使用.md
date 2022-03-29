# 概念
本质上，webpack 是一个用于现代 JavaScript 应用程序的 静态模块打包工具。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 依赖图(dependency graph)，然后将你项目中所需的每一个模块组合成一个或多个 bundles，它们均为静态资源，用于展示你的内容。

# 直接上手
**前置条件**
在学习webpack之前，请先保证你的环境有
* node
* npm
## 构建webpack的文件目录
**1.初始化配置**
创建一个webpack-learn1文件夹并进入然后运行命令
```node
npm init
```
会问你各种构建文件的问题，一路回车就行了
**2.安装webpack和webpack-cli**
在命令行运行,-D 是开发时使用的
```node
npm i -D webpack@4.41.6
npm i -D webpack-cli@3.3.11
```
**
**3.创建项目结构**
- 1.在当前项目下创建src文件夹
- 2.在src文件夹下创建data.json，保存数据
如：
```json
    {
        "name": "shuan",
        "age": "22"
    }
```
- 3.src文件夹下创建index.js
```js
function add (a, b) {
    return a + b
}
console.log(add(a, b))
```

- 4.在src文件夹下创建index.css
```css
html,body {
    background: 'pink'
}
```

-5.在命令行运行
```node
webpack ./src/index.js -o ./build/built.js --mode=development
```
```node
webpack ./src/index.js -o ./build/built.js --mode=production
```

index.js: webpack入口起点文件

**运行指令：**
-  开发环境：
  webpack ./src/index.js -o ./build/built.js --mode=development


    webpack会以 ./src/index.js 为入口文件开始打包，打包后输出到 ./build/built.js
    整体打包环境，是开发环境

-  生产环境：
  webpack ./src/index.js -o ./build/built.js --mode=production


    webpack会以 ./src/index.js 为入口文件开始打包，打包后输出到 ./build/built.js
    整体打包环境，是生产环境

**结论：**
 1. webpack能处理js/json资源，不能处理css/img等其他资源
 2. 生产环境和开发环境将ES6模块化编译成浏览器能识别的模块化~
 3. 生产环境比开发环境多一个压缩js代码。