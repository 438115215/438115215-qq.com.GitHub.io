### Object 类型

ECMAScript 中的对象其实就是一组数据和功能的集合。对象通过 new 操作符后跟对象类型的名称
来创建。开发者可以通过创建 Object 类型的实例来创建自己的对象，然后再给对象添加属性和方法：
```js
let o = new Object(); 
```
这个语法类似 Java，但 ECMAScript 只要求在给构造函数提供参数时使用括号。如果没有参数，如
上面的例子所示，那么完全可以省略括号（不推荐）：
```js
let o = new Object; // 合法，但不推荐
```
Object 的实例本身并不是很有用，但理解与它相关的概念非常重要。类似 Java 中的 java.lang. 
Object，ECMAScript 中的 Object 也是派生其他对象的基类。Object 类型的所有属性和方法在派生
的对象上同样存在。
每个 Object 实例都有如下属性和方法。
* constructor：用于创建当前对象的函数。在前面的例子中，这个属性的值就是 Object() 
函数。
* hasOwnProperty(propertyName)：用于判断当前对象实例（不是原型）上是否存在给定的属
性。要检查的属性名必须是字符串（如 o.hasOwnProperty("name")）或符号。
* isPrototypeOf(object)：用于判断当前对象是否为另一个对象的原型。（第 8 章将详细介绍
原型。）
* propertyIsEnumerable(propertyName)：用于判断给定的属性是否可以使用（本章稍后讨
论的）for-in 语句枚举。与 hasOwnProperty()一样，属性名必须是字符串。
* toLocaleString()：返回对象的字符串表示，该字符串反映对象所在的本地化执行环境。
* toString()：返回对象的字符串表示。
* valueOf()：返回对象对应的字符串、数值或布尔值表示。通常与 toString()的返回值相同。
因为在 ECMAScript 中 Object 是所有对象的基类，所以任何对象都有这些属性和方法。第 8 章将
介绍对象间的继承机制。