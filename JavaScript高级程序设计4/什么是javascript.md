## 简短历史回顾
随着web越来越流行，对客户端脚本语言的需求也越来越强烈。当时大多数用户使用28.8kbit/s的调制解调器上网，但是网页越来越复杂，为了检验简单表单需要大量与服务器通讯成为用户痛点

1995年，网景公司的 Brendan Eich开始为即将发布的Netscape navigator2开发了一个叫Mocha后来叫LiveScript的脚本语言。本来是计划在客户端和服务端都是用它，它在服务器叫LiveWire

为了赶发布，sun和网景合作开发，在发布前改名JavaScript

由于JavsScript1.0很成功，微软向IE投入更多资源，不久就发布了IE3其中包括了JScript的JavsScript实现。1996年8月微软冲入web浏览器领域，JavaScript作为一门语言向前迈进了一大步。

微软的JavsScript实现意味着出现了两个版本的JavScript
* Netscape Navigator的JavaScript
* IE 的JScript
与C语言和其他编程语言不同，JavaScript还没有规范其语法或特性的标准，两个版本的并存让这个问题更加突出。随着业界的担忧，JavsScript终于走上了标准化的征程。

1997年，JavaScript1.1作为提案被提交给欧洲计算机制造商协会ECMA。
第39技术委员会TC39承担了‘标准化一门通用、跨平台、厂商中立的脚本语言的语法和语义’的任务（参考TC39-ECMAScript）。

1998年，国际标准化组织ISO和国际电工委员会IEC将ECMAScript采纳为标准ISO/IEC-16262。自此以后各家浏览器均采用ECMAScript作为自己实现JavaScript实现的依据，虽然实现各有不同

## JavaScript实现
完整的JavaScript包含一下几个部分

* ECMAScript 核心
* DOM 文档对象模型
* BOM 浏览器对象模型

### ECMAScript
ECMAScript 并不局限与浏览器，没有输入输出方法。ECMA-262将这门语言作为一个基准来定义，以便在它之上再构建更稳健的脚本语言。web浏览器只是ECMAScript实现可能存在的一种环境。环境提供ECMAScript的基准实现和环境自身交互必须的拓展。其他的环境还有node、Adobe Flash等

不涉及浏览器的话，ECMA-262到底定义了什么？在基本的层面，它描述这门语言如下部分：

* 语法
* 类型
* 语句
* 关键字
* 保留字
* 操作符
* 全局对象

## 小结
Javascript是一门用来与网页交互的脚本语言包含一下三个组成部分
 * ECMAScript: 由ECMA-262定义并提供核心功能
 * 文档对象模型（DOM）：提供与网页内容交互的方法和接口
 * 浏览器对象模型（BOM）：提供与浏览器交互的方法和接口