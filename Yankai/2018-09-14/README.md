# Javscript的垃圾回收机制与内存泄露

## 简介

像C语言这样的高级语言一般都有底层的内存管理接口，比如 `malloc()` 和 `free()`。另一方面，JavaScript创建变量（对象，字符串等）时分配内存，并且在不再使用它们时“自动”释放。 后一个过程称为垃圾回收。这个“自动”是混乱的根源，大多数的Javascript开发者并没有过多的关注这一点。

## 内存生命周期

无论什么程序语言，内存生命周期基本是一致的：

1. 分配你所需要的内存
2. 使用分配到的内存（读，写）
3. 不需要的时候将其释放

基本所有语言中第二条都是明确的。第一和第三部分在底层语言中也是明确的。但是在Javascript这种高级语言中，大部分都是隐含的。

### Javascript中的内存分配

#### 值的初始化化

Javscript在定义变量时就自动完成了内存分配

```js
var n = 123  //给数值分配内存
var s = "lorem"  //给字符串分配内存

var o = {
  a: 1,
  b: null
}  //给对象和其包含的值分配内存

function f(a) {
  return a + 12
} //给函数（可调用的对象）分配内存

//函数表达式也能分配一个对象
someElement.addEventListener('click', function() {
  someElement.style.backgroundColor = 'blue';
}, false)
```

#### 通过函数调用分配内存

有些函数的调用结果是分配对象内存：

```js
var d = new Date()  //分配一个Date对象
var e = document.createElement('div')  //分配一个DOM元素
```

有些方法分配了新变量或者新对象：

```js
var f = "lorem";
var f2 = f.substr(0, 3);  //f2是一个新字符串
// 因为字符串是不变量
// Javascript可能决定不分配内存
// 只是存储了 [0-3] 的范围

var arr = ['string1', 'string2'];
var arr2 = ['string3', 'string2'];
var arr3 = arr.concat(arr2);
// 新数组有四个元素，是 arr 与 arr2 合并的结果
```

### 使用值

使用值的过程实际上是对分配内存进行读取与写入的操作。读取与写入可能是写入一个变量或者一个对象的属性值，甚至传递函数的参数。

### 当内存不再需要时释放

大多数内存管理的问题都在这个阶段。在这里最艰难的任务是找到“所分配的内存确实已经不再需要了”。其他语言往往需要开发者自行确定哪一块内存不再需要并释放它。

高级语言解释器嵌入了“垃圾回收器”，它的主要工作是跟踪内存的分配和使用，以便当分配的内存不再使用时，自动释放它。这只能是一个近似的过程，因为要知道是否仍然需要某块内存是无法判定的（无法通过某种算法解决）。

## 垃圾回收

垃圾回收算法主要依赖于引用的概念。在内存管理的环境中，一个对象如果有访问另一个对象的权限（隐式或者显式），叫做一个对象引用另一个对象。例如，一个Javascript对象具有对它原型的引用（隐式引用）和对它属性的引用（显式引用）。

在这里，“对象”的概念不仅特指 JavaScript 对象，还包括函数作用域（或者全局词法作用域）。

### 引用计数垃圾收集

这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。


```js
var o = { 
  a: {
    b:2
  }
}; 
// 两个对象被创建，一个作为另一个的属性被引用，另一个被分配给变量o
// 很显然，没有一个可以被垃圾收集


var o2 = o; // o2变量是第二个对“这个对象”的引用

o = 1;      // 现在，“这个对象”的原始引用o被o2替换了

var oa = o2.a; // 引用“这个对象”的a属性
// 现在，“这个对象”有两个引用了，一个是o2，一个是oa

o2 = "yo"; // 最初的对象现在已经是零引用了
           // 他可以被垃圾回收了
           // 然而它的属性a的对象还在被oa引用，所以还不能回收

oa = null; // a属性的那个对象现在也是零引用了
           // 它可以被垃圾回收了
```
#### 存在问题：循环引用导致的内存泄漏

IE 6, 7 使用引用计数方式对 DOM 对象进行垃圾回收。该方式常常造成对象被循环引用时内存发生泄漏

```js
var div;
window.onload = function(){
  div = document.getElementById("myDivElement");
  div.circularReference = div;
  div.lotsOfData = new Array(10000).join("*");
};
```

上面的例子中，`myDivElement` 这个 DOM 元素里的 `circularReference` 属性引用了 `myDivElement`，造成了循环引用。如果该属性没有显示移除或者设为 null，引用计数式垃圾收集器将总是且至少有一个引用，并将一直保持在内存里的 DOM 元素，即使其从DOM 树中删去了。如果这个 DOM 元素拥有大量的数据 (如上的 `lotsOfData` 属性)，而这个数据占用的内存将永远不会被释放。

### 标记-清除算法

这个算法把“对象是否不再需要”简化定义为“对象是否可以获得”。

这个算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象。

这个算法比前一个要好，因为“有零引用的对象”总是不可获得的，但是相反却不一定，参考“循环引用”。

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法。所有对JavaScript垃圾回收算法的改进都是基于标记-清除算法的改进，并没有改进标记-清除算法本身和它对“对象是否不再需要”的简化定义。

#### 针对上面循环引用的例子

用标记-清除算法，一旦div和其事件处理无法从根获取到，他们将被垃圾回收器回收。

## 常见的几种内存泄漏

### 意外的全局变量

js中如果不用 var 声明变量,该变量将被视为全局对象的属性,也就是全局变量.

```js
function foo(arg) {
  bar = "this is a hidden global variable";
}
```

这种内存泄漏可以用Javascript的 `use strict` 避免，严格模式会组织你意外创建的全局变量。

### 被遗忘的定时器或者回调

```js
var someResource = getData();
setInterval(function() {
  var node = document.getElementById('Node');
  if(node) {
    node.innerHTML = JSON.stringify(someResource));
  }
}, 1000);
```

这样的代码很常见, 如果 id 为 `Node` 的元素从 DOM 中移除, 该定时器仍会存在, 同时, 因为回调函数中包含对 `someResource` 的引用, 定时器外面的 `someResource` 也不会被释放.

### 没有清理的DOM元素引用

```js
var elements = {
  button: document.getElementById('button'),
  image: document.getElementById('image'),
  text: document.getElementById('text')
};
 
function doStuff() {
  image.src = 'http://some.url/image';
  button.click();
  console.log(text.innerHTML);
}
 
function removeButton() {
  document.body.removeChild(document.getElementById('button'));
    
  // 虽然我们用removeChild移除了button, 但是还在elements对象里保存着#button的引用
  // 换言之, DOM元素还在内存里面.
}
```

### 闭包

```js
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var someMessage = '123';
  var unused = function () {
    if (originalThing)
      console.log("hi");
  };
  theThing = {
    someMethod: function () {
      longStr: new Array(1000000).join('*'),
      console.log(someMessage);
    }
  };
};
```

这个代码片段做了一件事：每次 `replaceThing` 被调用的时候，`theThing` 获取到一个包括一个大数组和新闭包(`someMethod`)的新对象。同时，变量 `unused` 保留了一个有 `originalThing`（`theThing` 从之前的对 `replaceThing ` 的调用）引用的闭包。重要的是一旦一个作用域被在同个父作用域下的闭包创建，那个作用域是共享的。这种情况下，为闭包 `somMethod ` 创建的作用域被 `unused` 共享了。`unused` 有一个对 `originalThing ` 的引用。即使 `unused` 从来没被用过，`someMethod` 可以通过 `theTing` 被使用。由于 `someMethod` 和 `unused` 共享了闭包作用域，即使 `unused` 从来没被用过，它对 `originalThing` 的引用迫使它停留在活跃状态（不能回收）。当这个代码片段重复运行的时候，可以看到内存使用稳步的增长。GC运行的时候，这并不会减轻。本质上，一组关联的闭包被创建（同 `unused` 变量在表单中的根节点一起），这些闭包作用域中每个带了大数组一个非直接的引用，导致了大型的泄漏。

## 总结

内存泄露只会在少数情况下会发生，并且在大多数情况下是可以被避免的，合理的利用Chrome的Devtool可以很容易检测到内存泄露的位置。

## 参考

[https://blog.csdn.net/z_Sherry/article/details/53353551](https://blog.csdn.net/z_Sherry/article/details/53353551)

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)

[https://blog.csdn.net/ikscher/article/details/38517069](https://blog.csdn.net/ikscher/article/details/38517069)





























