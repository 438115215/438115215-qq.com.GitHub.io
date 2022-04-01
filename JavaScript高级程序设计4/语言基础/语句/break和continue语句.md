### break 和 continue 语句

break 和 continue 语句为执行循环代码提供了更严格的控制手段。其中，break 语句用于立即退
出循环，强制执行循环后的下一条语句。而 continue 语句也用于立即退出循环，但会再次从循环顶部
开始执行。下面看一个例子：
```js
let num = 0; 
for (let i = 1; i < 10; i++) { 
 if (i % 5 == 0) { 
 break;
  } 
 num++; 
} 
console.log(num); // 4 
```
在上面的代码中，for 循环会将变量 i 由 1 递增到 10。而在循环体内，有一个 if 语句用于检查 i
能否被 5 整除（使用取模操作符）。如果是，则执行 break 语句，退出循环。变量 num 的初始值为 0，
表示循环在退出前执行了多少次。当 break 语句执行后，下一行执行的代码是 console.log(num)，
显示 4。之所以循环执行了 4 次，是因为当 i 等于 5 时，break 语句会导致循环退出，该次循环不会执
行递增 num 的代码。如果将 break 换成 continue，则会出现不同的效果：
```js
let num = 0; 
for (let i = 1; i < 10; i++) { 
 if (i % 5 == 0) { 
 continue; 
 } 
 num++; 
} 
console.log(num); // 8 
```
这一次，console.log 显示 8，即循环被完整执行了 8 次。当 i 等于 5 时，循环会在递增 num 之
前退出，但会执行下一次迭代，此时 i 是 6。然后，循环会一直执行到自然结束，即 i 等于 10。最终
num 的值是 8 而不是 9，是因为 continue 语句导致它少递增了一次。
break 和 continue 都可以与标签语句一起使用，返回代码中特定的位置。这通常是在嵌套循环中，
如下面的例子所示：
```js
let num = 0; 
outermost: 
for (let i = 0; i < 10; i++) { 
 for (let j = 0; j < 10; j++) { 
 if (i == 5 && j == 5) { 
 break outermost; 
 } 
 num++; 
 } 
} 
console.log(num); // 55 
```
在这个例子中，outermost 标签标识的是第一个 for 语句。正常情况下，每个循环执行 10 次，意
味着 num++语句会执行 100 次，而循环结束时 console.log 的结果应该是 100。但是，break 语句带
来了一个变数，即要退出到的标签。添加标签不仅让 break 退出（使用变量 j 的）内部循环，也会退出
（使用变量 i 的）外部循环。当执行到 i 和 j 都等于 5 时，循环停止执行，此时 num 的值是 55。continue
语句也可以使用标签，如下面的例子所示：
```js
let num = 0; 
outermost: 
for (let i = 0; i < 10; i++) { 
 for (let j = 0; j < 10; j++) {
     if (i == 5 && j == 5) { 
 continue outermost; 
 } 
 num++; 
 } 
} 
console.log(num); // 95 
```
这一次，continue 语句会强制循环继续执行，但不是继续执行内部循环，而是继续执行外部循环。
当 i 和 j 都等于 5 时，会执行 continue，跳到外部循环继续执行，从而导致内部循环少执行 5 次，结
果 num 等于 95。
组合使用标签语句和 break、continue 能实现复杂的逻辑，但也容易出错。注意标签要使用描述
性强的文本，而嵌套也不要太深。