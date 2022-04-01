### Boolean 类型

Boolean (布尔值)类型是ECMAScript中使用最频繁的类型之一，有两个字面值: true和false。

这两个布尔值不同于数值，因此true不等于1，false不等于0。下面是给变量赋布尔值的例子: 
```js
let found = true;
let lost =false;
```

注意布尔值字面量true和false是区分大小写的，因此True和False (及其他大小混写形式)是有效的标识符，但不是布尔值。虽然布尔值只有两个但所有其他ECMAScript类型的值都有相应布尔值的等价形式。要将一个其他类型的值转换为布尔值，可以调用特定的Boolean()转型函数:

```js
let message = "Hello world!";
let messageAsBoolean = Boolean(message);
```
在这个例子中，字符串message会被转换为布尔值并保存在变量messageAsBoolean中。Boolean()转型函数可以在任意类型的数据上调用，而且始终返回一个布尔值。什么值能转换为true或false的规则取决于数据类型和实际的值。下表总结了不同类型与布尔值之间的转换规则。
* 数据类型 - 转化为true的值 - 转化为false的值
* Boolean - true - false
* String - 非空字符串 - ""(空字符串)
* Number - 非零数组(包括无穷值) - 0、NaN
* Object - 任意对象 - null
* Undefined - N/A(不存在) - undefined

理解以上转换非常重要，因为像if等流控制语句会自动执行其他类型值到布尔值的转换，例如:

```js
let message = "Hello world!";
if (message) {
    console.log("Value is true");
}
```
在这个例子中，console.log 会输出字符串"Value is true"，因为字符串message会被自动转换为等价的布尔值true。由于存在这种自动转换，理解流控制语句中使用的是什么变量就非常重要。错误地使用对象而不是布尔值会明显改变应用程序的执行流。