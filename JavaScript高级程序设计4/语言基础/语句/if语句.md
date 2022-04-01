### if 语句
if 语句是使用最频繁的语句之一，语法如下：
```js
if (condition) statement1 else statement2 
```
这里的条件（condition）可以是任何表达式，并且求值结果不一定是布尔值。ECMAScript 会自
动调用 Boolean()函数将这个表达式的值转换为布尔值。如果条件求值为 true，则执行语句
statement1；如果条件求值为 false，则执行语句 statement2。这里的语句可能是一行代码，也可
能是一个代码块（即包含在一对花括号中的多行代码）。来看下面的例子：
```js
if (i > 25) 
 console.log("Greater than 25."); // 只有一行代码的语句
else { 
 console.log("Less than or equal to 25."); // 一个语句块
} 
```
这里的最佳实践是使用语句块，即使只有一行代码要执行也是如此。这是因为语句块可以避免对什
么条件下执行什么产生困惑。
可以像这样连续使用多个 if 语句：
```js
if (condition1) statement1 else if (condition2) statement2 else statement3 
```
下面是一个例子：
```js
if (i > 25) { 
 console.log("Greater than 25."); 
} else if (i < 0) { 
 console.log("Less than 0."); 
} else { 
 console.log("Between 0 and 25, inclusive."); 
}
```