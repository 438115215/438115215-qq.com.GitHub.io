### while 语句
while 语句是一种先测试循环语句，即先检测退出条件，再执行循环体内的代码。因此，while 循
环体内的代码有可能不会执行。下面是 while 循环的语法：
```js
while(expression) statement 
```
这是一个例子：
```js
let i = 0; 
while (i < 10) { 
 i += 2; 
} 
```
在这个例子中，变量 i 从 0 开始，每次循环递增 2。只要 i 小于 10，循环就会继续。