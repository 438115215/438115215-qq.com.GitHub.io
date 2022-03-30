# HTML中的Javascript

## \<script\>元素

将JavaScript插入HTML使用\<script\>元素
其存在下列8个属性

* async：可选的，表示应该立即开始下载脚本，不能阻止其他页面动作如（下载资源、等待其他脚本加载），只对外部脚本文件有效
* charset：可选，使用src指定代字符集
* crossorigin：可选，配置相关的CORS（跨域资源共享）
* defer：可选，表示脚本可以延迟到文档完全被解析和显示之后再执行
* integrity：可选，允许比对接收到的资源的指定的加密签名来验证子资源完整性
* language：废弃
* src：可选，表示外部的地址
* type：可选，代替language，表示代码块中脚本语言的内容类型也叫MIME类型

使用\<scrip\>的方式有2种
* 网页中嵌入js代码
* 使用src外部引入

要在行内嵌入js代码，直接把代码放在\<script\>元素中就行
```html
<script>
    function sayHi() {
        console.log("Hi!");
    }
</script>
```
包含在\<script\>中的代码会被从上到下解释。
在\<script\>元素中的代码被计算完成之前，页面的其余内容不会加载，也不会显示。

要包含外部文件中的JavsScript，就必须使用src属性，值是URL，指向包含JavaScript代码的文件
```html
<script src="example.js"></script>
```
在解释外部JavaScript文件时，页面也会阻塞。浏览器在解析这个资源时，会向src属性的值发送一个GET请求。这个请求不受浏览器同源策略限制，但返回并被执行的JavaScript受限制。

### 标签位置
过去\<script\>标签都放在\<head\>标签内,这种做法目的是，把外部CSS和JS文件集中放置，不过JS文件放在这里，必须把所有JS下载、解析、解释后才能渲染页面，会造成一段时间的页面空白，为了解决这个问题通常放置在\<body\>元素后面

### 异步执行脚本
async属性，不保证js文件出现的次序执行

### 动态执行脚本
使用DOM API
```js
let script = document.createElement('script');
script.src = 'gibberish.js';
document.head.appendChild(script);
```
**注意**

在元素添加到HTML之前不会发送请求，默认异步加载。但是不是所有浏览器都支持async属性，所以可以明确设置同步加载
```js
let script = document.createElement('script');
script.src = 'gibberish.js';
script.async = false;
document.head.appendChild(script);
```
这种方法获取的资源对浏览器与加载器是不可见的，会严重影响它们在资源获取队列中的优先级。根据应用程序的工作方式一级怎么使用，这种方式可能会严重影响性能，要想让预加载知道这些动态请求文件的存在，可以在文档头部显式声明它们：
```html
<link rel="preload" href="gibberish.js">
```

### XHTML中的变化
XHTML是将HTML作为XML的应用重新包装的结果。在XHTML中使用JavsScript必须指定type值为text/javascirpt

XHTML编写代码比HTML中严格，如下面代码在HTML中有效，但XHTML无效
```html
<script type="text/javascript">
    function compare(a, b) {
        if (a < b) {
            console.log("A < B")
        } else if ( a > b ) {
            console.log("A > B")
        } else {
            console.log("A = B")
        }
    }
</script>
```
在HTML`<script>`元素会应用特殊规则。XHTML则没有。这意味着`<`会被解释成一个标签的开始，标签开始 的后面不能有空格，这回导致语法错误。
## 行内代码与外部文件
虽然html文件可以嵌入js代码，但是通常认为最佳实现是js代码在外部文件中。推荐理由如下：
* 可维护性
* 缓存
* 适应未来

## 文档模式
文档模式既可以使用doctype切换文档模式。
最初的文档模式有两种：

**混杂模式**
让IE像IE5一样，支持一些非标准特性,在所有浏览器中都以省略文档开头的doctype声明作为开关。

**标准模式**
让IE具有兼容标准行为，标准模式通过下列几种文档类型声明开启
```html
<!--HTML 4.01 Strict -->
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 //EN" "https://www.w3.org/TR/html4/strict.dtd">
<!--XHTML 1.0 Strict -->
<! DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "https://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<!--HTML 5 -->
<! DOCTYPE html>
```

**准标准模式**
是后来加的，支持很多标准的特性，但是没有标准规定的严格。主要区别在于如何对待图片元素周围的空白.准标准模式通过过渡性文档类型和框架集文档类型来触发
```html
<! -- HTML 4.01 Transitional -->
<! DOCTYPE HTML PUBLIC -//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<! -- HTML 4.01 Frameset -->
<! DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd" >
<! -- XHTML 1.0 Transitional -->
<! DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtm1-transitional.dtd">
<! -- XHTML 1.0 Frameset -->
<! DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1 .0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
```

## \<nosrcipt\>元素
早期浏览器不支持JavaScript，需要一个页面优雅降级的处理方案。`<nosrcipt>`被用于不支持js的浏览器提供了替代内容。

`<noscript>`元素可以包含任何可以出现在`<body>`中的HTML元素，`<script>`除外。
在下列两种情况下，浏览器将显示包含在 `<noscript>`中的的内容:
* 浏览器不支持脚本
* 浏览器对脚本的支持被关闭

任何一个条件被满足，被包含在`<noscript>`中的内容就会被渲染，否则不会

```js
<! DOCTYPE html>
<html>
<head>
<title> Example HTML Page </title>
<script defer= "defer" src="example1.js" >
</script>
<script defer= "defer" src="example2.js"></script>
</head>
<body>
<noscript>
<p> This page requires a JavaScript-enabled browser. </p>
</noscript>
</body>
</html>
```
这个例子是在脚本不可用时让浏览器显示一段话。如果浏览器支持脚本，则用户永远不会看到它。

## 小结
JavaScript是通过`<scrip>`元素插入到HTML页面中的。这个元素可用于把JavaScript代码嵌入到HTML页面中跟其他标记混合在一起，也可用于引入保存在外部文件中的JavaScript. 本章的重点可以总结如下。
* 要包含外部JavaScript文件，必须将src属性设置为要包含文件的URL。文件可以跟网页在同一台服务器上，也可以位于完全不同的域。
* 所有`<script>`元素会依照它们在网页中出现的次序被解释。在不使用defer和async属性的情况下，包含在`<script>`元素中的代码必须严格按次序解释。
* 对不推迟执行的脚本，浏览器必须解释完位于`<script>`元素中的代码，然后才能继续渲染页面的剩余部分。为此，通常应该把`<script>`元素放到页面末尾，介于主内容之后及`</body>`标签之前。
* 可以使用defer属性把脚本推迟到文档渲染完毕后再执行。推迟的脚本原则上按照它们被列出的次序执行。
* 可以使用async属性表示脚本不需要等待其他脚本，同时也不阻塞文档渲染，即异步加载。异步脚本不能保证按照它们在页面中出现的次序执行。
* 通过使用`<noscript>`元素，可以指定在浏览器不支持脚本时显示的内容。如果浏览器支持并启用脚本，则`<noscript>`元素中的任何内容都不会被渲染。