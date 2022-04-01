### with 语句
with 语句的用途是将代码作用域设置为特定的对象，其语法是：
```js
with (expression) statement; 
```
使用 with 语句的主要场景是针对一个对象反复操作，这时候将代码作用域设置为该对象能提供便
利，如下面的例子所示：
```js
let qs = location.search.substring(1); 
let hostName = location.hostname; 
let url = location.href; 
上面代码中的每一行都用到了 location 对象。如果使用 with 语句，就可以少写一些代码：
with(location) { 
 let qs = search.substring(1); 
 let hostName = hostname; 
 let url = href; 
} 
```
这里，with 语句用于连接 location 对象。这意味着在这个语句内部，每个变量首先会被认为是
一个局部变量。如果没有找到该局部变量，则会搜索 location 对象，看它是否有一个同名的属性。如
果有，则该变量会被求值为 location 对象的属性。
严格模式不允许使用 with 语句，否则会抛出错误