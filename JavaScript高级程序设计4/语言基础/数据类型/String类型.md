### String类型

String (字符串)数据类型表示零或多个16位Unicode字符序列。字符串可以使用双引号(")、单引号(') 或反引号()标示，因此下面的代码都是合法的: 

```js
let firstName = "John"; 
let lastName = 'Jacob';
let lastName = `Jingleheimerschmidt`
```

跟某些语言中使用不同的引|号会改变对字符串的解释方式不同，ECMAScript语法中表示字符串的引|号没有区别。不过要注意的是，以某种弓|号作为字符串开头，必须仍然以该种引号作为字符串结尾。比如，下面的写法会导致语法错误:

```js
let firstName = 'Nicholas"; 
//语法错误:开头和结尾的引号必须是同一种
```

**1.字符字面量**

字符串数据类型包含一些字符字面量， 用于表示非打印字符或有其他用途的字符，如下所示:

| 字面量      | 含义 |
| ----------- | ----------- |
| \n      | 换行       |
| \t   | 制表        |
| \b   | 退格        |
| \r   | 回车        |
| \f   | 换页        |
| \\\   | 反斜杠(\)        |
| \\'   | 单引号( ' ),在字符串以单引号标示时使用，例如'He said, \'hey.\"'        |
| \\"   | 双引号("),在字符串以双引号标示时使用，例如"He said, \"hey. \""        |
| \\`   | 反引号(、),在字符串以反引号标示时使用，例如'He said, \"hey.\"        |
| \xnn   | 以十六进制编码nn表示的字符(其中n是十六进制数字0~F ),例如\x41等于"A"        |
| \unnnn   | 以十六进制编码nnnn表示的Unicode字符(其中n是十六进制数字0~F ),例如\u03a3等于希腊字符"∑"        |

这些字符字面量可以出现在字符串中的任意位置，且可以作为单个字符被解释:
```js
let text = "This is the letter sigma: \u03a3.";
```

在这个例子中，即使包含6个字符长的转义序列，变量text仍然是28个字符长。因为转义序列表示一个字符，所以只算一个字符。

字符串的长度可以通过其length属性获取:

```js
console.log(text.length); // 28
```

这个属性返回字符串中1 6位字符的个数。

注意如果字符串中包含双字节字符，那么length属性返回的值可能不是准确的字符数

**2.字符串的特点**

ECMAScript中的字符串是不可变的\(immutable),意思是一旦创建,它们的值就不能变了。要修改某个变量中的字符串值，必须先销毁原始的字符串，然后将包含新值的另一个字符串保存到该变量， 如下所示:

```js
let lang = "Java";
lang = lang + "Script"; 
```

这里，变量lang开始包含字符串"Java"。紧接着,lang被重新定义为包含"Java"和"Script”的 组合，也就是"JavaScript"。整个过程首先会分配一个足够容纳10个字符的空间，然后填充上"Java" 和"Script"。最后销毁原始的字符串"Java"和字符串"Script"，因为这两个字符串都没有用了。所有处理都是在后台发生的，而这也是一些早期的浏览器(如Firefox 1.0之前的版本和IE6.0)在拼接字符串时非常慢的地解决了这个问题。

**3.转化为字符串**

有两种方式把一个值转换为字符串。 首先是使用几乎所有值都有的toString(方法。这个方法唯一-的用途就是返回当前值的字符串等价物。比如:
```js
let age = 11;
let ageAsString = age.toString0;//字符串11
let found = true; 
let foundAsString = found.toString0; //字符串true
```

toString(方法可见于数值、布尔值、对象和字符串值。(没错， 字符串值也有toString(方法，该方法只是简单地返回自身的一一个副本。) null和undefined值没 有toString()方法。多数情况下，toString(不接收任何参数。不过，在对数值调用这个方法时，toString()可以接收一个底数参数，即以什么底数来输出数值的字符串表示。默认情况下，toString()返回数值的十进制字符串表示。而通过传入参数，可以得到数值的二进制、八进制、十六进制，或者其他任何有效基数的字符串表示，比如:

```js
let num = 10;
console.log(num.toString());//"10"
console.log(num.toString(2));//"1010"
console.log(num.tosting(8)); //"12"
console.log(num.toString(10);//"10"
console.log(num.toString(16));// "a'
```

这个例子展示了传入底数参数时，toString()输出的字符串值也会随之改变。数值10可以输出为任意数值格式。注意，默认情况下(不传参数)的输出与传入参数10得到的结果相同。如果你不确定一个值是不是null或undefined，可以使用String()转型函数，它始终会返回表示相应类型值的字符串。String()函数遵循如下规则。

* 如果值有toString()方法，则调用该方法(不传参数)并返回结果。
* 如果值是null,返回"null" 。
* 如果值是undefined,返回"undefined"。

下面看几个例子:

```js
let value1 = 10;
let value2 = true;
let value3 = null;
let value4;
console.log(String(value1)); // "10"
console.log(String(value2)); // "true"
console.log(String(value3)); // "null"
console.log(String(value4)); // "undefined"
```

这里展示了将4个值转换为字符串的情况:一个数值、一个布尔值、一个 null和一个undefined。数值和布尔值的转换结果与调用toString()相同。因为null和undefined没有toString()方法，所以String(方法就直接返回了这两个值的字面量文本。

注意用加号操作符给一个值加上一个空字符串""也可以将其转换为字符串

**4.模板字面量**

ECMAScript 6新增了使用模板字面量定义字符串的能力。与使用单引号或双引号不同，模板字面量保留换行字符，可以跨行定义字符串:

```js
let myMultiLineString = 'first line\nsecondline';
let myMultiLineTemplateLiteral = `first line second line `;
console.log(myMultiLineString);
// first line
// second line"
console.log(myMultiLineTemplateLiteral);
// first line
// second line"
console.log(myMultiLineString === myMultiLinetemplateLiteral); // true
```

顾名思义，模板字面量在定义模板时特别有用，比如下面这个HTML模板:

```js
let pageHTML = `
<div>
<a href="#">
<span> Jake </span>
</a>
</div> 
`;
```

由于模板字面量会保持反引号内部的空格，因此在使用时要格外注意。格式正确的模板字符串看起来可能会缩进不当: 

```js
//这个模板字面量在换行符之后有25个空格符
let myTemplateLiteral = `first line second line`;
console.log(myTemplateLiteral.length); //47
//这个模板字面量以一个换行符开头
let secondTemplateLiteral =`first line
second line `;
console.log(secondTemplateLiteral[0] ==='\n'); // true
//这个模板字面量没有意料之外的字符
let thirdTemplateLiteral = `first line
second line `;
console.log(thirdTemplateLiteral);
// first line
// second line
```

**5.字符串插值**


模板字面量最常用的一个特性是支持字符串插值，也就是可以在一个连续定义中插入一个或多个值。技术上讲，模板字面量不是字符串，而是一种特殊的JavaScript句法表达式，只不过求值后得到的是字符串。模板字面量在定义时立即求值并转换为字符串实例，任何插入的变量也会从它们最接近的作用域中取值。

字符串插值通过在${}中使用一个JavaScript表达式实现:

```js
let value = 5;
let exponent = 'second';
//以前，字符串插值是这样实现的:
let interpolatedString = value + 'to the' + exponent + ' power is'+ (value * value);
//现在，可以用模板字面量这样实现:
let interpolatedTemplateLiteral =`${ value } to the ${ exponent } power is ${ value * value }`;
console.log(interpolatedString);
//5 to the second power is 25
console.log(interpolatedTemplateLiteral);
// 5 to the second power is 25
```

所有插入的值都会使用toString()强制转型为字符串，而且任何JavaScript表达式都可以用于插值。嵌套的模板字符串无须转义:

```js
console.log(`Hello, ${ `World` }!` ); // Hello,World!
```

将表达式转换为字符串时会调用toString():
```js
let foo = { toString:() => 'World' };
console.log(`Hello, ${ foo }!`); // Hello,World!
```

在插值表达式中可以调用函数和方法:
```js
function capitalize(word) {
    return `${ word[0].toUpperCase() }${word.slice(1)}`;}

console.log(`${ capitalize('hello') }, ${capitalize('world')}!` ); // Hello, World!
```

此外，模板也可以插入自己之前的值:

```js
let value = '';
function append() {
    value = `${value}abc`
    console.log(value);
    }
    append(); // abc
    append(); // abcabc
    append(); // abcabcabc
```

**6.模板字面量标签函数**

模板字面量也支持定义标签函数(tag function) ,而通过标签函数可以自定义插值行为。标签函数会接收被插值记号分隔后的模板和对每个表达式求值的结果。

标签函数本身是一一个常规函数，通过前缀到模板字面量来应用自定义行为，如下例所示。标签函数接收到的参数依次是原始字符串数组和对每个表达式求值的结果。这个函数的返回值是对模板字面量求值得到的字符串。

最好通过一个例子来理解:
```js
let a= 6;
let b= 9;
function simpleTag(strings, aValExpression, bValExpression, sumExpression) {
    console.log(strings);
    console.log(aValExpression);
    console.log(bValExpression);
    conssole.log(sumExpression);
    return 'foobar';
    }
let untaggedResult= `${a}+ ${b}= ${a + b}`;
let taggedResult=simpleTag`${a}+${b}=${a+b}`;
//["","+ "," ="," ]
// 6
// 9
// 15
console.log(untaggedResult); // "6+9=15"
console.log(taggedResult);// "foobar"
```

因为表达式参数的数量是可变的，所以通常应该使用剩余操作符(rest operator)将它们收集到一个数组中:

```js
let a = 6;
let b = 9;
function simpleTag(string,...expressions){
    console.log(strings);
    for(const expression of expressions) {
        console.log(expression);
        }
    return 'foobar';
}
let taggedResult = simpleTag`${a} + ${b} = ${a + b}`;
//["","+ "," ="," ]
// 6
// 9
// 15
console.log(taggedResult); // "foobar"
```


对于有n个插值的模板字面量，传给标签函数的表达式参数的个数始终是n，而传给标签函n+1。因此，如果你想把这些字符串和对表达式求值的结果拼接起来作为默认返回的字符串，可以这样做: 

```js
let a = 6;
let b = 9;
function zipTag(string,...expressions){
    return strings[0] + expressions.map((e, i) => `${e}${sting[i + 1]}`).join('');
}
let untaggedResult = `${a} + ${b} = ${a + b}`;
let taggedResult = zipTag`${a} + ${b} = ${a + b}`;
console.log(untaggedResult); // 6 + 9 = 15
console.log(taggedResult); // 6 + 9 = 15
```

**7.原始字符串**

使用模板字面量也可以直接获取原始的模板字面量内容(如换行符或Unicode字符)，而不是被转换后的字符表示。为此，可以使用默认的String.raw标签函数:
```js
// Unicode示例
// \u00A9是版权符号
console.log(`\u00A9`);//
console.log(String.raw `\u00A9`); // \u00A9
//换行符示例
console.log(`first line\nsecond line`);
// first line
// second line
console.log(String.raw `first line\nsecondline` ); // "first line\nsecond line'
//对实际的换行符来说是不行的
//它们不会被转换成转义序列的形式
console.log(`first line
second line` );
// first line
// second line
console.log(String.raw `first line
second line `);
// first line
// second line
```
另外，也可以通过标签函数的第一个参数， 即字符串数组的. raw属性取得每个字符串的原始内容:
```js
function printRaw(strings) {
console.log(" Actual characters:");
for (const string of strings) {
console.log(string);
}
console.log('Escaped characters; );
for (const rawString of strings.raw) {
    console.log(rawString);
    }
}
printRaw`\u00A9${ 'and' }\n`;
// Actual characters:
// c
// (换行符)
// Escaped characters:
// \u00A9
// \n
```