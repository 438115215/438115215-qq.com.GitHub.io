### for 语句
for 语句也是先测试语句，只不过增加了进入循环之前的初始化代码，以及循环执行后要执行的表
达式，语法如下：
```js
for (initialization; expression; post-loop-expression) statement
```
下面是一个用例：
```js
let count = 10; 
for (let i = 0; i < count; i++) { 
 console.log(i); 
} 
```
以上代码在循环开始前定义了变量 i 的初始值为 0。然后求值条件表达式，如果求值结果为 true
（i < count），则执行循环体。因此循环体也可能不会被执行。如果循环体被执行了，则循环后表达式
也会执行，以便递增变量 i。for 循环跟下面的 while 循环是一样的：
```js
let count = 10; 
let i = 0; 
while (i < count) { 
 console.log(i); 
 i++; 
} 
```
无法通过 while 循环实现的逻辑，同样也无法使用 for 循环实现。因此 for 循环只是将循环相关
的代码封装在了一起而已。
在 for 循环的初始化代码中，其实是可以不使用变量声明关键字的。不过，初始化定义的迭代器变
量在循环执行完成后几乎不可能再用到了。因此，最清晰的写法是使用 let 声明迭代器变量，这样就可
以将这个变量的作用域限定在循环中。
初始化、条件表达式和循环后表达式都不是必需的。因此，下面这种写法可以创建一个无穷循环：
```js
for (;;) { // 无穷循环
 doSomething(); 
} 
```
如果只包含条件表达式，那么 for 循环实际上就变成了 while 循环：
```js
let count = 10; 
let i = 0; 
for (; i < count; ) { 
 console.log(i); 
 i++; 
} 
```
这种多功能性使得 for 语句在这门语言中使用非常广泛。