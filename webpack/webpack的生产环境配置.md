# webpack的生产环境配置

## 提取CSS成单独文件

**项目结构**
--src
----css
------a.css
------b.css
----js
------index.js
----index.html
--webpack.config.js

**a.css**
```css
#box1 {
  width: 100px;
  height: 100px;
  background-color: pink;
}
```
**b.css**
```css
#box2 {
  width: 200px;
  height: 200px;
  background-color: deeppink;
}
```
**index.js**
```js
import '../css/a.css';
import '../css/b.css';
```
**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>webpack</title>
</head>
<body>
  <div id="box1"></div>
  <div id="box2"></div>
</body>
</html>
```
**webpack.config.js**
```js
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/js/index.js',
  output: {
    filename: 'js/built.js',
    path: resolve(__dirname, 'build')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 创建style标签，将样式放入
          // 'style-loader', 
          // 这个loader取代style-loader。作用：提取js中的css成单独文件
          MiniCssExtractPlugin.loader,
          // 将css文件整合到js文件中
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    }),
    new MiniCssExtractPlugin({
      // 对输出的css文件进行重命名
      filename: 'css/built.css'
    })
  ],
  mode: 'development'
};
```
