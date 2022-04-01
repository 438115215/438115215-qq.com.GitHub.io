### switch 语句
switch 语句是与 if 语句紧密相关的一种流控制语句，从其他语言借鉴而来。ECMAScript中 switch
语句跟 C 语言中 switch 语句的语法非常相似，如下所示：
```js
switch (expression) { 
 case value1: 
 statement
 break; 
 case value2: 
 statement 
 break; 
 case value3: 
 statement 
 break; 
 case value4: 
 statement 
 break; 
 default: 
 statement 
} 
```
这里的每个 case（条件/分支）相当于：“如果表达式等于后面的值，则执行下面的语句。”break
关键字会导致代码执行跳出 switch 语句。如果没有 break，则代码会继续匹配下一个条件。default
关键字用于在任何条件都没有满足时指定默认执行的语句（相当于 else 语句）。
有了 switch 语句，开发者就用不着写类似这样的代码了：
```js
if (i == 25) { 
 console.log("25"); 
} else if (i == 35) { 
 console.log("35"); 
} else if (i == 45) { 
 console.log("45"); 
} else { 
 console.log("Other"); 
} 
```
而是可以这样写：
```js
switch (i) { 
 case 25: 
 console.log("25"); 
 break; 
 case 35: 
 console.log("35"); 
 break; 
 case 45: 
 console.log("45"); 
 break; 
 default: 
 console.log("Other"); 
} 
```
为避免不必要的条件判断，最好给每个条件后面都加上 break 语句。如果确实需要连续匹配几个
条件，那么推荐写个注释表明是故意忽略了 break，如下所示：
```js
switch (i) { 
 case 25: 
 /*跳过*/ 
 case 35: 
 console.log("25 or 35"); 
 break; 
 case 45: 
 console.log("45"); 
 break; 
 default:
 console.log("Other"); 
} 
```
虽然 switch 语句是从其他语言借鉴过来的，但 ECMAScript 为它赋予了一些独有的特性。首先，
switch 语句可以用于所有数据类型（在很多语言中，它只能用于数值），因此可以使用字符串甚至对象。
其次，条件的值不需要是常量，也可以是变量或表达式。看下面的例子：
```js
switch ("hello world") { 
 case "hello" + " world": 
 console.log("Greeting was found."); 
 break; 
 case "goodbye": 
 console.log("Closing was found."); 
 break; 
 default: 
 console.log("Unexpected message was found."); 
} 
```
这个例子在 switch 语句中使用了字符串。第一个条件实际上使用的是表达式，求值为两个字符串
拼接后的结果。因为拼接后的结果等于 switch 的参数，所以 console.log 会输出"Greeting was 
found."。能够在条件判断中使用表达式，就可以在判断中加入更多逻辑：
```js
let num = 25; 
switch (true) { 
 case num < 0: 
 console.log("Less than 0."); 
 break; 
 case num >= 0 && num <= 10: 
 console.log("Between 0 and 10."); 
 break; 
 case num > 10 && num <= 20: 
 console.log("Between 10 and 20."); 
 break; 
 default: 
 console.log("More than 20."); 
} 
```
上面的代码首先在外部定义了变量 num，而传给 switch 语句的参数之所以是 true，就是因为每
个条件的表达式都会返回布尔值。条件的表达式分别被求值，直到有表达式返回 true；否则，就会一
直跳到 default 语句（这个例子正是如此）。