# Javascript原型链和instanceof运算符之间的暧昧关系

## 面试题
```js
Function instanceof Function
String instanceof String
```

## 关于Javascript原型链

在Javascript语言中，整个核心的体系结构都是围绕着两个构造函数：`Object` 和 `Function` 构建的，这里引用一张 [mollypages.org](http://www.mollypages.org/) 的Javascript对象体系图来说明。

![img](http://pchbeel8i.bkt.clouddn.com/20180816-1.jpg)

每个函数默认都会有一个属性叫 `prototype` ，每个通过函数和 `new` 操作符生成的对象都具有一个属性叫 `__proto__`，这个属性保存了创建它的构造函数的 `prototype` 属性的引用。这个 `__proto__` 对象就是实现原型链的核心对象。

## 关于instanceof

### `typeof` 与 `instanceof`

平时我们判断数据类型会比较常用 `typeof` 运算符，它可以判断出 `number`, `boolean`, `string`，但是对于 `object` 的判断能力比较弱，例如 `Array` 和 `null` 以及使用 `new` 操作符创建的对象判断结果均为 `object`。

```js
typeof [1, 2, 3]   //--->'object'
typeof null       //--->'object'
typeof new Number(233)   //--->'object'
typeof new String('lorem')   //--->'object'
```

这种情况就需要 `instanceof` 运算符出马了。

```js
[1,2,3] instanceof Array   //--->true
null instanceof Object   //--->false
new Number(233) instanceof Number   //--->true
new String('lorem') instanceof String   //--->true
```
以上是 `instanceof` 的基本用法：判断 `a` 是否是 `b` 类型。

但是从MDN上对 `instanceof` 的解释看来，这个运算符还有其他的用法。

> `instanceof` 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 `prototype` 属性。

也就是说 `instaneof` 还可以判断父类型。

```js
function Father() {}
function Child() {}
Child.prototype = new Father();
var a = new Child();
console.log(a instanceof Child);  // true
console.log(a instanceof Father); // true
```

`instanceof` 运算符的内核可以简单的用下面的代码描述

```js
function check(a, b) {
  while(a.__proto__) {
    if(a.__proto__ === b.prototype)
      return true;
    a = a.__proto__;
  }
  return false;
}
```

简单的说，a如果是b的实例，那么a肯定能使用b的 `prototype` 中定义的方法和属性，那么用代码表示就是a的原型链中有 `b.prototype` 取值相同的对象，于是顺着a的原型链一层层找就行了。

## 回到最开始的两道题

直接用伪代码解析

`Function instanceof Function`

```js
// 区分左右两边的表达式
FunctionLeft = Function, FunctionRight = Function;

// 根据规范逐步推演
O = FunctionRight.prototype = Function.prototype;
L = FunctionLeft.__proto__ = Function.prototype;

0 == L
// return true
```

`String instanceof String`

```js
// 区分左右两边的表达式
StringLeft = String, StringRight = String;

O = StringRight.prototype = String.prototype;
L = StringLeft.__proto__ = Function.prototype;

// 首次判断
O != L

// 开始查找 L 是否还有 __proto__
L = Function.prototype.__proto__ = Object.prototype;

// 第二次判断
O != L

// 再次查找 L 是否还有 __proto__
L = Object.prototype.__proto__ = null

// 第三次判断
L == null

// return false

```

#### 题外话
上面的代码可以看到 `null instanceof Object` 返回了 `false`，是因为 `null` 在Javascript中用来表示空引用的一个特殊值，本质上 `null` 和 `Object` 不是一个数据类型，`null` 值并不是以 `Object` 为原型创建出来的，所以返回了 `false`。

而为什么 `typeof null` 会返回 `object`。在MDN上有这样一段解释：

> 在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null的类型标签也成为了 0，typeof null就错误的返回了"object"。

属于一个历史遗留问题。

#### 一个小坑

在使用 `instanceof` 运算符判断 `true instanceof Boolean` 会返回 `false`。是由于Javscript的数据类型导致的，`true` 所属的 `Boolean` 类型属于Javascript的基础类型，使用引用类型的 `new Boolean(true) instanceof Boolean` 才会返回期望的结果 `true`, 所以要判断Boolean类型的值，还是推荐使用 `typeof`。

#### 参考
[https://www.cnblogs.com/zichi/p/4561564.html](https://www.cnblogs.com/zichi/p/4561564.html)

[https://www.cnblogs.com/objectorl/archive/2010/01/11/Object-instancof-Function-clarification.html](https://www.cnblogs.com/objectorl/archive/2010/01/11/Object-instancof-Function-clarification.html)

[https://www.cnblogs.com/SourceKing/p/5766210.html](https://www.cnblogs.com/SourceKing/p/5766210.html)

[https://blog.csdn.net/u014181418/article/details/51720008](https://blog.csdn.net/u014181418/article/details/51720008)

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)






























