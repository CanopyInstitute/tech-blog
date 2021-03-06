# 为什么 ['1', '7', '11'].map(parseInt) 返回 [1, NaN, 3]


> 作者：alentan
> 链接：https://juejin.im/post/5d0202da51882546dd10087b
> 来源：掘金

我们试着用数组自带的 `map` 函数将数组中的每一个值通过 `parseInt` 从字符串转换为整数，但是却出现了意料之外的结果

```js
['1', '7', '11'].map(parseInt)  //[1, NaN, 3]
```

![img](https://s2.ax1x.com/2019/06/23/ZPJCBq.png)

我们最终得到的不是一个 `[1, 7, 11]` 的整数数组， 而是一个奇怪的 `[1, NaN, 3]` 数组，要了解到底发生了什么，我们先要讨论一些Jacscript的基础概念。

## 真值&假值

Javascript中最简单的 `if-else` 语句

```js
if (true) {
    // 永远都会运行
} else {
    // 永远不会运行
}
```

在这种情况下，`if-else` 语句的条件为 `true`，因此始终执行 if-block 语句块并忽略 else-block 语句块。这是一个简单的例子，因为 `true` 是一个布尔值。如果我们将非布尔值作为条件会怎么样呢?

```js
if ("hello world") {
    // 他会运行吗?
    console.log("Condition is truthy");
} else {
    // 还是运行这个?
    console.log("Condition is falsy");
}
```

尝试在浏览器的控制台中运行此代码（Chrome上的F12）。您应该发现 if-block 语句块运行。这是因为字符串对象"hello world"是 `true`。

每个Javascript对象当放置在布尔上下文中时都是真值或假值。例如 if-else 语句，会把Javascript对象转为 `true` 或 `false`。那么哪些对象是true的，哪些是false的呢？这是一个简单的规则：

Javascript中当以下的值放置在布尔上下文中时会返回false:

* `false `
* `0（""空字符串）`
* `null`
* `undefined`
* `NaN`

## 进制

```
0 1 2 3 4 5 6 7 8 9 10
```

我们从0到9计数时，每个数字（0-9）都有不同的符号。但是，一旦我们达到10，我们需要两个不同的符号（1和0）来表示数字。这是因为我们的小数计数系统的基数（或进制）为10。
基数是最小的数字，只能由多个符号表示。不同的计数系统具有不同的基数，因此，相同的数字可以指计数系统中的不同数字。

|10进制|2进制|16进制|
|---|---|---|
0    |     0|         0|
1    |     1 |        1|
2    |     10 |       2|
3   |      11  |      3|
4   |      100  |     4|
5   |      101   |    5|
6   |      110    |   6|
7   |      111    |   7|
8    |     1000    |  8|
9   |      1001     | 9|
10  |      1010     | A|
11 |       1011     | B|
12 |       1100     | C|
13 |       1101   |   D|
14 |       1110   |   E|
15 |       1111    |  F|
16 |       10000   |  10|
17|        10001   |  11|

例如，查看上表，我们看到相同的数字11可以表示不同计数系统中的不同数字。如果进制（基数）为2，则表示数字 3。如果进制（基数）为16，则表示数字17。

您可能已经注意到，在我们的示例中，当输入为11时，parseInt返回3，这对应于上表中的二进制列。

## 函数参数

Javascript中的函数可以使用任意数量的参数调用，即使它们不等于声明的函数参数的数量。缺少的参数被视为未定义，而多余的参数会被忽略（但会存储在类似数组的 `arguments` 中）。

```js
function foo(x, y) {
    console.log(x);
    console.log(y);
}
foo(1, 2);      // logs 1, 2
foo(1);         // logs 1, undefined
foo(1, 2, 3);   // logs 1, 2
```

## map()函数

`Map` 是`Array` 原型中的一个方法，它返回一个新的数组，其结果是将原始数组的每个元素传递给一个函数。例如，以下代码将数组中的每个元素乘以3

```js
function multiplyBy3(x) {
    return x * 3;
}
const result = [1, 2, 3, 4, 5].map(multiplyBy3);
console.log(result);   // logs [3, 6, 9, 12, 15];
```

现在，假设我想使用map()（没有返回语句）记录每个元素。我应该能够console.log作为一个参数传递给map()......

```js
[1,2,3,4,5] .map（console.log）;
```

![img](https://s2.ax1x.com/2019/06/23/ZPJPH0.png)

发生了一些奇怪的事情，每次console.log调用都记录索引和完整数组，而不是仅记录值。

```js
[1, 2, 3, 4, 5].map(console.log);
// 上面的例子相当于
[1, 2, 3, 4, 5].map(
    (val, index, array) => console.log(val, index, array)
);
// 而不是这样
[1, 2, 3, 4, 5].map(
    val => console.log(val)
);
```

当一个函数传递到map()，对于每次迭代，三个参数传递到函数：currentValue，currentIndex，和完整的array。这就是每次迭代记录三个条目的原因。

我们现在拥有解决这个谜团所需的所有部分。

## 揭开谜团

`ParseInt` 有两个参数：`string`和`radix`（进制）。如果提供的`radix`（进制）为空或者为假值，进制（基数）默认设置为10。

```js
parseInt('11');                => 11
parseInt('11', 2);             => 3
parseInt('11', 16);            => 17
parseInt('11', undefined);     => 11 (radix（进制） 为假)
parseInt('11', 0);             => 11 (radix（进制） 为假)
```

现在让我们一步一步地运行我们的示例。

```js
['1', '7', '11'].map(parseInt);       => [1, NaN, 3]
// 第一次迭代: val = '1', index = 0, array = ['1', '7', '11']
parseInt('1', 0, ['1', '7', '11']);   => 1
```

由于0是假的，因此进制（基数）设置为默认值10。 parseInt()只接受两个参数，因此['1', '7', '11']会被 忽略。字符串'1'在10进制（基数）中的字符串表示数字1。

```js
// 第二次迭代: val = '7', index = 1, array = ['1', '7', '11']
parseInt('7', 1, ['1', '7', '11']);   => NaN
```

在进制（基数）为1系统中，符号'7'不存在。与第一次迭代一样，忽略最后一个参数。所以，parseInt()返回了NaN。

```js
// 第三次迭代: val = '11', index = 2, array = ['1', '7', '11']
parseInt('11', 2, ['1', '7', '11']);   => 3
```

在进制（基数）为2（二进制）系统中，符号'11'表示数字3，最后一个参数被忽略。

## 总结

['1', '7', '11'].map(parseInt) 不能按预期工作，因为在每次迭代中 map 传递三个参数到 parseInt()。第二个参数 index 作为radix（进制）参数传递给 parseInt 。因此，使用不同的进制（基数）解析数组中的每个字符串。'7' 被解析为进制（基数）为 1，它是 NaN，'11' 被解析为进制（基数）为 2，它的值为 3，'1' 被解析为默认的进制（基数）为 10，因为它的索引 0 是假值。


























