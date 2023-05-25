# Liquid
## Basic
### 简介
Liquid 代码可分为 `对象（object）`、`标记（tag）` 和 `过滤器（filter）`。

---
#### 对象

对象 告诉 Liquid 在页面的哪个位置展示内容。对象和变量名由双花括号标识：{{ 和 }}。

* 输入

```js
{{ page.title }}
```
* 输出

```js 
Introduction
```
上述实例中，Liquid 输出 page.title 对象的内容，此对象保存的是文本 Introduction。

---

#### 标记（tag）

标记（tag） 创造了模板的逻辑和控制流。他们由单括号加百分号标识：{% 和 %}。

标记（tag）并不产生任何可见的文本输出。这意味着你可以用他们为变量赋值、创建条件和循环逻辑，并且不在页面上显示出任何 Liquid 逻辑代码。

* 输入

```js
{% if user %}
  Hello {{ user.name }}!
{% endif %}
```

* 输出
```js
Hello Adam!
```

标记被分为三类：

* 控制流
* 迭代
* 变量赋值
你可以在每一类标记所对应的章节了解更多信息。

---
#### 过滤器

过滤器 改变 Liquid 对象的输出。他们被用在输出上，通过一个 | 符号分隔。


- 输入


```js
{{ "/my/fancy/url" | append: ".html" }}
```

* 输出

```js
/my/fancy/url.html
```

多个过滤器可以共同作用于同一个输出，并按照从左到右的顺序执行。

- 输入

```js
{{ "adam!" | capitalize | prepend: "Hello " }}
```

- 输出

```js
Hello Adam!
```

---
### 操作符

Liquid 包含了大量逻辑（logical）和比较操作符（comparison operator）。

---
#### 基本操作符

|-|-|
|-|-|
|==|	相等|
|!=|	不相等|
|>	|大于|
|<	|小于|
|>=|	大于或等于|
|<=|	小于或等于|
|or|	逻辑或|
|and|	逻辑与|

例如：

```
{% if product.title == "Awesome Shoes" %}
  These shoes are awesome!
{% endif %}
```

可以在一个标记（tag）中使用多个操作符：

```
{% if product.type == "Shirt" or product.type == "Shoes" %}
  This is a shirt or a pair of shoes.
{% endif %}
```

---
#### contains（包含）

contains 用于检查在一个字符串中是否存在某个子串。

```
{% if product.title contains 'Pack' %}
  This product's title contains the word Pack.
{% endif %}
```

contains 还可以用于检查一个字符串数组中是否存在某个字符串。

```
{% if product.tags contains 'Hello' %}
  This product has been tagged with 'Hello'.
{% endif %}
```

contains 只能用于搜索字符串。你不能将其用于从一个对象数组中检查是否存在某个对象。

### 真值与假值

编程时，在条件判断中任何返回 true 的都被叫做 真值（truthy）。
任何返回 false 的都被叫做 假值（falsy）。
所有的对象（object）类型都可以被描述为真值（truthy）或假值（falsy）。

* Truthy
* Falsy
* Summary
  
---
#### 真值（Truthy）
除了 `nil` 和 `false` 之外的所有值都是真值。

如下例，字符串 “Tobi” 虽不是布尔类型，但是其在条件判断时被当做真值（truthy）：

```
{% assign tobi = "Tobi" %}

{% if tobi %}
  This condition will always be true.
{% endif %}
```

`字符串（String）`，即便是空字符串，也是真值（truthy）。

如下例，如果 settings.fp_heading 是个空字符串将会输出空 HTML 标签：

* 输入

```
{% if settings.fp_heading %}
  <h1>{{ settings.fp_heading }}</h1>
{% endif %}
```

* 输出
```
<h1></h1>
```
---

#### 假值（Falsy）
在 Liquid 中，nil 和 false 是假值。

* 总结
下表总结了在 Liquid 中什么是真值什么是假值。
|-|-|-|
|-|-|-|
 |-	|真值（truthy）|	假值（falsy）|
|true|	•	| |
|false	| |	•|
|nil	| |	•|
|string	|•	| |
|empty string|	•	| |
|0	|•	 |
|integer|	•|	 |
|float|	•	| |
|array	|•	| |
|empty array|	•	| |
|page	|•	| |
|EmptyDrop|	•|	 |

### 数据类型

Liquid 对象的类型可以是以下五种：

* String
* Number
* Boolean
* Nil
* Array


你可以通过 assign 或 capture 标记来初始化 Liquid 变量。

* String（字符串）
将变量的值包裹在单引号或双引号之中就声明了一个字符串：

```
{% assign my_string = "Hello World!" %}
```

* Number（数字）
数字类型包括浮点数和整数：

```
{% assign my_int = 25 %}
{% assign my_float = 39.756 %}
```

* Boolean（布尔）
Booleans 类型只能是 true 或 false。布尔值千万不能加引号，否则就成为字符串了。

```
{% assign foo = true %}
{% assign bar = false %}
```

* Nil（空）
Nil 是一个特殊的空值，当 Liquid 代码没有可输出的结果时将返回 Nil。他并不是由 “nil” 这个三个字符组成的字符串。

在 if 条件判断和其他 Liquid 标记（tag）判断语句中，Nil 被当做 false 。

下例中，如果 user 不存在（也就是 user 返回 nil），Liquid 不输出问候语：

```
{% if user %}
  Hello {{ user.name }}!
{% endif %}
```

如果 Liquid 标记（tag）或输出返回的是 nil，页面上将不会有任何内容。

* 输入

```
The current user is {{ user.name }}
```

* 输出

```
The current user is
Array（数组）
```

数组能够存储一组任意类型的变量。

访问数组中的项
通过 迭代标记（iteration tag） 可以访问数组中的所有项。

* 输入

```
<!-- if site.users = "Tobi", "Laura", "Tetsuro", "Adam" -->
{% for user in site.users %}
  {{ user }}
{% endfor %}
```

* 输出

```
Tobi Laura Tetsuro Adam
```

访问数组中的特定项
利用方括号 [ ] 能够访问数组中的特定项。数组的索引从 0 开始。

* 输入

```
<!-- if site.users = "Tobi", "Laura", "Tetsuro", "Adam" -->
{{ site.users[0] }}
{{ site.users[1] }}
{{ site.users[3] }}
```
* 输出

```
Tobi
Laura
Adam
```

初始化数组
你无法只通过 Liquid 语法初始化一个数组。

然而，你可以利用 split 过滤器将一个字符串分割为一个子字符串数组。

### Liquid的各种分支

Liquid 是一门灵活、安全的模版语言，被用于许多不同环境中。Liquid 被创建之初是用在 Shopify 商店系统中的，后来也被广泛用于 Jekyll 网站中。随着时间的推移，Shopify 和 Jekyll 分别为 Liquid 添加了针对各自用途的对象（object）、标记（tag）和过滤器（filter）。目前最流行的 Liquid 版本包括 Liquid、Shopify Liquid 和 Jekyll Liquid。

本站点托管的是最新版本的 Liquid 的文档，包括了 beta 和 release candidate 版本中包含的特性，也就是说，是独立于 Shopify 和 Jekyll 之外的 Liquid。如果你是从 Liquid 仓库下载的代码或者安装的的是 gem 包，你所选择的 Liquid 版本对应你能够访问的对象（object）、标记（tag）和过滤器。

#### Shopify

Shopify 一直采用的都是最新版本的 Liquid，并且 Shopify 会针对 merchants’ store 为 Liquid 添加大量的对象（object）、标记（tag）和过滤器。这些新增的内容包括代表商店（store）、产品（product）和顾客信息的对象，以及用于展示商店数据和操作产品照片的过滤器。

Shopify 版本的 Liquid 所对应的文档在 Shopify Help Center。如果你希望尝试 Shopify 版本的 Liquid，你可以试用 Shopify 或者使用类似 DropPen 的工具。

#### Jekyll

Jekyll 是一个静态网站生成器，一个用于将模版和内容合并到一起从而创建网站的命令行工具。ekyll 将 Liquid 作为自身的模版语言，并且添加了许多对象（object）、标记（tag）和过滤器（filter）。这些新增内容包括代表内容页面的对象、用于在页面中引入内容片段的标记（tag），以及用于操作字符串和 URL 的过滤器。

Jekyll 还是 GitHub Pages 的底层引擎。GitHub Pages 是一项网站托管服务，允许你将 Jekyll 网站推送到 GitHub 仓库，最终得到一个发布到公网的站点。本网站就是由 GitHub Pages 托管的。

Jekyll 可能使用的不是最新版本的 Liquid。也就意味着本文档所列出的标记（tag）和过滤器不能在 Jekyll 中使用。通常 Jekyll 项目使用的是稳定版的 Liquid，而不使用 beta 或 release candidate 版本。通过 Jekyll 的 gem 信息也 可查看 Jekyll 所依赖的所有 gem 包，从而可以了解 Jekyll 所使用的 Liquid 版本。

Jekyll 版本的 Liquid 的文档在 Templates section of Jekyll’s documentation。如果你希望尝试 Jekyll 版本的 Liquid，你可以克隆 Jekyll 项目或者安装 Jekyll 的 gem 包，然后在静态网站中测试 Liquid。

### 控制输出的空白符

在 Liquid 模版中，你可以将连字符放在标记（tag）中，例如 {{-、-}}、{%- 和 -%}，用于将标记（tag）渲染之后的输出内容的左侧或右侧的空拍符剔除。

通常，即使不输出文本，模版中的任何 Liquid 表达式仍然会在渲染之后输出的 HTML 中包含一个空行：

* 输入

```
{% assign my_variable = "tomato" %}
{{ my_variable }}
```

请注意渲染之后输出的 “tomato” 字符前面包含了一个空行：
* 输出

```
tomato
```
通过为 assign 标记（tag）添加连字符，可以将渲染之后所输出的空拍符删除：

* 输入

```
{%- assign my_variable = "tomato" -%}
{{ my_variable }}
```
* 输出

```
tomato
```

如果你不希望任何标记（tag）被渲染之后所输出的内容有任何空白符，只需在所有标记（tag）两侧全部添加连字符即可，例如 ({%- 和 -%})：

* 输入

```
{% assign username = "John G. Chalmers-Smith" %}
{% if username and username.size > 10 %}
  Wow, {{ username }}, you have a long name!
{% else %}
  Hello there!
{% endif %}
```
* 不做空白符控制的输出


```
  Wow, John G. Chalmers-Smith, you have a long name!
```
* 输入

```
{%- assign username = "John G. Chalmers-Smith" -%}
{%- if username and username.size > 10 -%}
  Wow, {{ username }}, you have a long name!
{%- else -%}
  Hello there!
{%- endif -%}
```
* 带有空白符控制的输出

```
Wow, John G. Chalmers-Smith, you have a long name!
```