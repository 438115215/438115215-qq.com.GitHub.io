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