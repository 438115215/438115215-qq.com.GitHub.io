### Undefined类型
Undefined类型只有一个值,就是特殊值undefined。当使用var或let声明了变量但没有初始化时，就相当于给变量赋予了undefined值:

```js
let message;
console.log(message == undefined); //ture
```

在这个例子中，变量message在声明的时候并未初始化。而在比较它和undefined的字面值时，两者是相等的。这个例子等同于如下示例:
```js
let message=undefined;
console.log(message == undefined); // true
```
这里，变量message显 式地以undefined来初始化。但这是不必要的，因为默认情况下，任何未经初始化的变量都会取得undefined值。

**注意**

一般来说，永远不用显式地给某个变量设置undefined值。字面值undefined主要用于比较，而且在ECMA-262第3版之前是不存在的。增加这个特殊值的目的就是为了正式明确空对象指针(null) 和未初始化变量的区别。

注意，包含undefined值的变 量跟未定义变量是有区别的。请看下面的例子:

```js
let message;// 这个变量被声明了，只是值为undefined
//确保没有声明过这个变量
// let age
console.log(message); // "undefined"
console.log(age); // 报错
```

在上面的例子中，第一-个console.log会指出变量message的值，即"undefined"。 而第二个console.log要输出一个未声明的变量age的值，因此会导致报错。对未声明的变量，只能执行一个有用的操作，就是对它调用typeof。(对未声明的变 量调用delete也不会报错，但这个操作没什么用，实际上在严格模式下会抛出错误。)在对未初始化的变量调用typeof时，返回的结果是"undefined"，但对未声明的变量调用它时，返回的结果还是"undefined",这就有点让人看不懂了。比如下面的例子:

```js
let message; //这个变量被声明了，只是值为undefined
//确保没有声明过这个变量
//let age
console.log(typeof message);//"undefined" 
console.log(age);//"undefined"
```
无论是声明还是未声明，typeof返回的都是字符串"undefined"。逻辑上讲这是对的，因为虽然严格来讲这两个变量存在根本性差异，但它们都无法执行实际操作。

**注意**

即使未初始化的变量会被自动赋予undefined值，但我们仍然建议在声明变量的同时进行初始化。

这样，当typeof返回"undefined"时，你就会知道那是因为给定的变量尚未声明，而不是声明了但未初始化。

undefined是一个假值。 因此，如果需要，可以用更简洁的方式检测它。不过要记住，也有很多其他可能的值同样是假值。所以-定要明确自己想检测的就是undefined这个字面值，而不仅仅是假值。

```js
let message; //这个变量被声明了，只是值为undefined
// age没有声明
if (message) {
    //这个块不会执行
}
if(!message) {
    //这个块会执行
}
if (age) {
    //这里会报错
}
```