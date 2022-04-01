### for-in 语句
for-in 语句是一种严格的迭代语句，用于枚举对象中的非符号键属性，语法如下：
```js
for (property in expression) statement 
```
下面是一个例子：
```js
for (const propName in window) { 
 document.write(propName); 
} 
```
这个例子使用 for-in 循环显示了 BOM 对象 window 的所有属性。每次执行循环，都会给变量
propName 赋予一个 window 对象的属性作为值，直到 window 的所有属性都被枚举一遍。与 for 循环
一样，这里控制语句中的 const 也不是必需的。但为了确保这个局部变量不被修改，推荐使用 const。
ECMAScript 中对象的属性是无序的，因此 for-in 语句不能保证返回对象属性的顺序。换句话说，所有可枚举的属性都会返回一次，但返回的顺序可能会因浏览器而异。
如果 for-in 循环要迭代的变量是 null 或 undefined，则不执行循环体。