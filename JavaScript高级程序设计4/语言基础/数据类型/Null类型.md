### Null类型
Null类型同样只有一个值，即特殊值null。 逻辑上讲，null值表示一个空对象指针，这也是给typeof传一个null会返回"object"的原因:

```js
let car = null;
console.log(typeof car);
```
在定义将来要保存对象值的变量时，建议使用null来初始化，不要使用其他值。

这样只要检查这个变量的值是不是null就可以知道这个变量是否在后来被重新赋予了一个对象的引用，比如:

```js
if(car != null) {
    //car是一个对象引用
}
```
undefined值是由null值派生而来的，因此ECMA-262将它们定义为表面上相等,如下面的例子所示:
```js
console.log(null == undefined); //true
```

用等于操作符(==) 比较null和undefined始终返回true。但要注意，这个操作符会为了比较而转换它的操作数

即使null和undefined有关系，它们的用途也是完全不一样的。如前所述，永远不必显式地将变量值设置为undefined。但null不是这样的。任何时候，只要变量要保存对象，而当时又没有那个对象可保存，就要用null来填充该变量。这样就可以保持null是空对象指针的语义，并进一步将其与undefined区分开来。

null是一个假值。 因此，如果需要，可以用更简洁的方式检测它。不过要记住，也有很多其他可能的值同样是假值。所以一定要明确自己想检测的就是null这个字面值，而不仅仅是假值。

```js
let message = null;
let age;
if (message) {
    //这个块不会执行
}
if (! message) {
    //这个块会执行
}
if(age) {
    //这个块不会执行
}
if!age) {
    //这个块会执行
}
```