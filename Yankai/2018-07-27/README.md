# Javascript中的null、undefined和NaN

Javascript的基本数据类型除了 `Number`、`String`、`Boolean` 和 `Object` 外，还有两个比较容易混淆的 `null`、`undefined`。

## null
`null` 表示一个指向不存在或者无效的对象或者地址引用。他是一个全局对象，也是Javascript的初始值之一。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%889.39.30.png)

否定 `null` 值返回 `true`，但将其与 `false` （或者 `true`）比较会返回 `false`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%889.57.48.png)

在基础的数学计算里，`null` 值将被转换为 `0`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%889.59.53.png)


## undefined

全局属性 `undefined` 表示原始值 `undefined`。它也是Javascript的原始数据类型之一。`undefined` 是全局作用域的一个变量。`undefined` 的初始值就是 `undefined`。一个没有被赋值的变量的类型是 `undefined`。如果方法或者是语句中操作的变量没有被赋值，就会返回 `undefined`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8810.06.44.png)

如果声明一个变量但没有声明它的值的时候，Javascript会默认给它赋值 `undefined`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8810.04.52.png)

如果在任何运算中用了 `undefined`，它会返回 `NaN` 的值。和 `null` 差不多，否定 `undefined` 返回 `true`，与 `true` 或者 `false` 比较则返回 `false`。

## null和undefined的区别

### 相似之处

* 被否定时，`null` 和 `undefined` 都是 `true`
* 代表不存在的东西

### 不同之处

* `null` 表示无，不存在的；`undefined` 表示未定义
* `null` 是一个对象；`undefined` 是一个数据类型
* 基本数学运算中，`null` 被当做 `0`，`undefined` 返回的则是 `NaN`。

### `null` PK `undefined`

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8811.12.01.png)

`null == undefined` 返回 `true` 是因为Javascript对两个值进行了类型转换。
`null === undefined` 返回 `false` 是因为这次没有进行类型转换，在比较的时候同时比较了两者的数据类型。
最后两个就比较简单了，`!null == !undefined` 和 `!null === !undefined`，上文提到过两者的否定都返回了 `true`，所以这两个语句就相当于 `true == true` 和 `true === true`了。

## NaN（Not a Number）

全局 `NaN` 属性是个表示非数字的值。

当要得到的值不是数字时会返回 `NaN`。比如用字符串和数字做运算时会返回 `NaN`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8810.28.15.png)

但是在某些情况下，你并不会如愿的得到这个值.

比如Javascript在遇到 `+` 和一个字符串时，会自动把后面的元素追加到字符串里。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8811.00.13.png)

如果数字和布尔值运算时，布尔值会隐形的被转换成 `1` 和 `0`。 `true` 转换成 `1`，`false` 转换成 `0`。

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8811.02.54.png)

然而，**`NaN` 本身也是一个数字！** <img src="http://pchbeel8i.bkt.clouddn.com/timg.jpeg" width="130px"/>

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8811.03.18.png)

那 `NaN == NaN` 吗，并不！

![img](http://pchbeel8i.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-26%20%E4%B8%8B%E5%8D%8811.03.40.png)

这货自己都不等于自己。所幸，我们有 `isNaN()` 这个函数来判断值是否为 `NaN`。

```js
let itsNotABloodyNumber = "text" - 23

let itsANumber = 123

isNaN(itsNotABloodyNumber)  //---> true

isNaN(itsANumber)           //---> false

```

## 总结一下

`null` 表示不存在的或者无效的对象或者地址引用。简单数学运算中会被转换为 `0`，他是一个全局对象。

`undefined` 是一个全局属性，原始值是 `undefined`。它代表有些东西没赋值，没定义。`undefined` 不能被转换成数字，在数学运算里，会返回 `NaN`。

`NaN` 代表一个不是数字的东西，尽管它自己是一个数字。不等于自身。要检查一个值是不是 `NaN` 的时候需要用 `isNaN()` 函数。

在部分场景中使用 `===` 可以避免一些Javascript隐式类型转换的坑。



























