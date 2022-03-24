# var let const 的区别
1.var声明的变量会挂在在window对象上，意思就是你可以在全局进行访问var声明的变量
2.let 声明的变量只能在当前作用域内进行访问
3.const 声明的变量只能在当前作用域进行访问，并且声明的是一个常量

**例子：**
```js
var a = 'shuhan'
console.log(a)
console.log(window.a) // 和上面一样
```

```js
let b = 'shuhan'
console.log(b)
console.log(window.b) //undefine
```

```js
const a =  'shuhan'
a = 'shuhan111'// 报错
console.log(a)
```
var 的这一特性，会造成 window 全局变量的污染。
**例子：**

```js   
var innerHeight = 100;
console.log(window.innerHeight); 
```

**var 声明的变量存在变量提升，let 和 const 声明的变量不存在变量提升**
**例子：**
```js
console.log(a) // 打印shuhan
var a = 'shuhan'
```

```js
console.log(b) // 报错
let b = 'shuhan'
```

```js
console.log(c) // 报错
const c = 'shuhan'
```

**var 声明不存在块级作用域，let 和 const 声明存在块级作用域**
**例子:**
```js
  {
    var a = 'a';
    let b = 'b';
    const c = 'c';
  }

console.log(a); // a
console.log(b); // 报错：Uncaught ReferenceError: b is not defined ==> 找不到b这个变量
console.log(c); // 报错：Uncaught ReferenceError: c is not defined ==> 找不到c这个变量  
```

**同一作用域下，var 可以重复声明变量，let 和 const 不能重复声明变量**
```js
var a = 'a';
var a = 'aa';
console.log(a); // aa
```

```js
let b = 'b';
let b = 'bb';
console.log(b); //报错：Uncaught SyntaxError: Identifier 'b' has already been declared  ==> 变量 b 已经被声明了
```

```js
const c = 'c';
const c = 'cc';
console.log(c); //报错：Uncaught SyntaxError: Identifier 'c' has already been declared  ==> 变量 c 已经被声明了
```
备注：可以看出：使用 let/const 声明的变量，不会造成全局污染。

**let 和 const 的暂时性死区（DTC）**
**例子1（正常）**
```js
const name = 'shuhan';

function foo() {
    console.log(name);
}

foo(); // 执行函数后，打印结果：shuhan
```
上方例子中， 变量 name 被声明在函数外部，此时函数内部可以直接使用。
**例子2（报错）**
```js
const name = 'shuhan';

function foo() {
    console.log(name);
    const name = 'hello';
}

foo(); // 执行函数后，控制台报错：Uncaught ReferenceError: Cannot access 'name' before initialization
```
代码解释：如果在当前块级作用域中使用了变量 name，并且当前块级作用域中通过 let/const 声明了这个变量，那么，声明语句必须放在使用之前，也就是所谓的 DTC（暂时性死区）。DTC 其实是一种保护机制，可以让我们养成良好的编程习惯。
**const：一旦声明必须赋值；声明后不能再修改**
```js
const a;
console.log(a); // 报错：Uncaught SyntaxError: Missing initializer in const declaration
```

**总结**
基于上面的种种区别，我们可以知道：var 声明的变量，很容易造成全局污染。以后我们尽量使用 let 和 const 声明变量吧。

**const 常量被修改**
我们知道：用 const 声明的变量无法被修改。但是：
* 如果用 const 声明基本数据类型，则无法被修改；
* 如果用 const 声明引用数据类型（即“对象”），这里的“无法被修改”指的是不能改变内存地址的引用；但对象里的内容是可以被修改的。
**例子1（不能修改）**
```js
const name = 'shuhan';
name = 'vae'; // 因为无法被修改，所以报错：Uncaught TypeError: Assignment to constant variable
```

**例子2（不能修改）**
```js
const obj = {
    name: 'shuhan',
    age: 28,
};

obj = { name: 'vae' }; // 因为无法被修改，所以报错：Uncaught TypeError: Assignment to constant variable
```


**例子3（可以修改）**

```js
const obj = {
    name: 'shuhan',
    age: 28,
};
obj.name = 'vae'; // 对象里的 name 属性可以被修改
```

因为 变量名 obj 是保存在栈内存中的，它代表的是对象的引用地址，它是基本数据类型，无法被修改。但是 obj 里面的内容是保存在堆内存中的，它是引用数据类型，可以被修改。