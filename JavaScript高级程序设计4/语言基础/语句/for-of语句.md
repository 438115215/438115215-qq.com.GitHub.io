### for-of 语句
for-of 语句是一种严格的迭代语句，用于遍历可迭代对象的元素，语法如下：
```js
for (property of expression) statement 
```
下面是示例：
```js
for (const el of [2,4,6,8]) { 
 document.write(el); 
} 
```
在这个例子中，我们使用 for-of 语句显示了一个包含 4 个元素的数组中的所有元素。循环会一直
持续到将所有元素都迭代完。与 for 循环一样，这里控制语句中的 const 也不是必需的。但为了确保
这个局部变量不被修改，推荐使用 const。
for-of 循环会按照可迭代对象的 next()方法产生值的顺序迭代元素。关于可迭代对象，本书将在
第 7 章详细介绍。
如果尝试迭代的变量不支持迭代，则 for-of 语句会抛出错误。