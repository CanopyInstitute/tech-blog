# 理解CSS的字体度量与行高

`line-height` 是CSS中比较简单的属性。几乎所有前端开发都相信自己已经完全理解它是如何工作以及如何使用它。但事实上并不如此。它其实很复杂，也是CSS中的难点之一，而且也是CSS中特性之一：**内联格式化上下文（inline formatting context）**。

`line-height` 可以设置一个带有长度单位的值或者无单位的值，但是它的默认值为 `normal`。在CSS中 `normal` 是什么？我们通常认为它是 `1` 或者 `1.2`。甚至CSS规范都不清楚是哪一个。没有单位的 `line-height` 是相对于 `font-size` 的，但问题是，`font-size: 100px;` 在使用不同 `font-family` 时表现的行为是不一样的，所以 `line-height` 总是相同或不同的吗?

来深入了解下相应的机制。

## 先来聊聊font-size

先来看一段简单的HTML代码，一个 `p` 标签包含了三个 `span` 标签，每个 `span` 都被设置了不同的 `font-family` 

```html
<style>
  p {width: 1200px;font-size: 100px;}
  .a { font-family: Arial;background: red; }
  .b { font-family: Gadugi;background: red;}
  .c { font-family: Verdana;background: red; }
</style>
<p>
  <span class="a">Ba</span>
  <span class="b">Ba</span>
  <span class="c">Ba</span>
</p> 
```

每个元素使用相同的 `font-size`，但使用不同的 `font-family`，但渲染出来的 `line-height` 是不同的：

![img](http://pchbeel8i.bkt.clouddn.com/20181115-0.png)


即使我们意识到这种行为，但还是不清楚为什么 `font-size:100px` 时元素的 `height` 不是 `100px` ？测量发现：`Arial` 字体的高度是 `112px` ， `Gadugi` 字体的高度是 `133px` 和 `Verdana` 字体的高度是 `122px` 。

虽然看起来有点奇怪，但是它是完全可预期的，主要还是 `font-family` 的原因。这就需要搞清楚它是如何工作的。

* 字体定义其em-square，每个字符将会绘制出自己的容器。这个正方形使用相对单位和生成一个 `1000` 单位。但它也可以是 `1024` ，`2048` 或者其他
* 根据其荐对单位，字体的度量可以根据一些设置（ascender,descender,capital height,x-height等）来决定。注意，有些值是em-square之外的值
* 在浏览器中，相对单位是用于缩放用来适应所需的 `font-size`

下面以 `Arial` 字体为例，用 `FontForge` 来查看这个字体的度量参数：

![img](http://pchbeel8i.bkt.clouddn.com/20181115-1.png)

![img](http://pchbeel8i.bkt.clouddn.com/20181115-2.png)

* em-square 为 `2048`
* 上升(ascender)是 `1854` 和下降（descender）是 `434` 。相同的测试下，浏览器使用HHead Ascent/Descent值（Mac）和Win Ascent/Descent值（Windows），这些值可能不同。我们还需要注意，Capital高度是 `1467` 和x-height的值是 `1062`

这意味着 `Arial` 字体在 `2048` 个单位的em-square使用了 `1854 + 434` 个单位，也就是说 个单位，也就是说 `font-size:100px` 的时候，其高度是 `(1854 + 434) / 2048 * 100 = 111.71875px` 。这个计算高度定义了元素内容高度（在这篇文章中其它部分引用这个术语**content-area**）。你可能想到是内容区域相当于 `background` 属性。

![img](http://pchbeel8i.bkt.clouddn.com/20181115-3.jpg)

我们也可以看出来，大写字母是 `72px` 高度（1467个单位）和小写字母（x-hegiht）是 `52px` 高度（1062个单位）。因此， `1ex = 49px` 和 `1em = 100px`，而不是 `112px`（值得庆幸的是，em是基于font-size计算，而不是height）。

### 延伸一个概念 line-box

在继续深入之前，先要了解这涉及到什么？当 `<p>` 元素呈现在屏幕上，它根据它的宽度可以有很多线。每一行是由一个或多个行内元素（HTML标签元素或匿名内联元素文本内容）组成，专业术语称为行盒（line-box）。line-box的高度是基于它的子元素高度的。浏览器为每个行内元素计算的高度都是line-box（子元素的最高点到最低点）。因此line-box的总高度足以包含所有子元素（默认情况下）。

> 每个HTML元素实际上是一个line-box的堆栈。如果你知道每个line-box的高度，实际上你就知道每个元素的高度。

如果我们把前面的HTML结构更新成

```html
<p>
    Lorem ipsum dolor sit amet.
    <span class="a">Ba</span>
	<span class="b">Ba</span>
	<span class="c">Ba</span>
	consectetur adipisicing elit.
</p>
``` 

它会生成三个 `line-box`
* 第一个和最后一个每个包含一个匿名内联元素（文本内容）
* 第二个包含了两个匿名内联元素和三个 `<span>`

![img](http://pchbeel8i.bkt.clouddn.com/20181115-4.jpg)

我们清楚的看到，第二个line-box明显比其他的line-box要更高，根据子元素的内容区域（content-area）计算得来，更具体地说，是使用了`content-area` 更高的字体。

困难的是line-box创建部分是我们无法看到的，也不是用CSS控制它。

## line-height

直到现在，我们介绍了两个概念：**content-area**和**line-box**。如果仔细阅读了前面的内容，你应该知道line-box的高度是根据子元素高度来计算，但并不是子元素的内容区域（content-area）的高度。这是有很大区别的。

尽管这听起来可能有些奇怪，内联元素有两个不同高度：内容区域（content-area）高度和虚拟区域（virtual-area）高度

* 内容区域高度是由字体来决定的（前面介绍过）
* 虚拟区域（virtual-area）高度是 `line-height` ，它的高度用于计算line-box的高度

![img](http://pchbeel8i.bkt.clouddn.com/20181115-5.jpg)

计算虚拟区域（virtual-area）和内容区域（content-area）高度差称为leading。leading添加在内容区域顶部，另一半添加在内容区域底部。因此，内容区域总是在虚拟区域的中间。

根据其计算值，line-height（virtual-area）相同情况下比content-area更高或更低。对于较小的virtual-area，leading是负值的line-box要比它的子元素更小。

还有其他的内联元素：

* 替代内联行内元素（`<img>`，`<input>`，`<input>`等）
* `inline-block` 元素
* 行内元素参与特定格式化上下文（如，Flexbox元素，和所有的Flex项目）

对于这些特定的行内元素，高度计算基于他们的 `height`、`margin` 和 `border` 属性。如果 `hegiht` 的值是 `auto`，然后使用 `line-height` 时content-area严格上等于 `line-height`。

![img](http://pchbeel8i.bkt.clouddn.com/20181115-6.jpg)

无论如何，我们仍然面临的问题是 `line-height` 的 `normal` 值是多小？答案是，其计算content-area高度还是依据于里面的字体来度量。

我们回到FontForge。Arial的em-square是2048，但我们看到ascender/descender的值：

生成的Ascent/Descent: ascender是1638，descender是410。用于绘制字符（OS/2）
度量的Ascent/Descent: ascender是1854，descender是434。用于内容区域高度(hhea和OS/2)
度量线的间距：通过Ascent/Descent度量使用 `line-height: normal`（hhea）
在我们的示例中，Catamaran字体定义了 `67` 个单位的线间距（Line Gap），因此 `line-height: normal` 的值将等于内容区域，也就是 `1150` 个单位或 `1.15`。

作为比较，Catamaran字体的一个em-square是1000个单位，其ascender是 `1100`，descender是 `540`，线间距是 `0`。这意味着，`font-size: 100px` 的内容区域是 `164px`（`1640` 个单位）和 `line-height` 是 `164px`（`1640` 个单位或 `1.64`）。所有这些度量都是特殊字型，由字体设计师来设置。

显而易见，设置 `line-height:1` 是一个非常糟糕的做法。`font-size` 没有单位的观念是相对的，但内容区域不是相对的以及处理虚拟区域小于内容区域有很多问题存在。

![img](http://pchbeel8i.bkt.clouddn.com/20181115-7.jpg)

据统计，在1117种常用字体中，有95%的字体计算的 `line-height` 大于1。而这些字体计算的 `line-height` 值的浮动区域在 `0.618` 到 `3.378`。

## 总结

* 行内格式化上下文真的很难理解
* 所有行内元素都有两个高度
* 内容区域（content-area）基于字体的度量参数
* 虚拟区域（virtual-area）就是 `line-height`
* 这两个高度是无法可视的（如果你通过开发者工具，你可以看到）
* `line-height:normal` 是基于字体度量参数
* `line-height: n` 有可能创建一个虚拟区域比内容区域更小

## 参考

[https://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align](https://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align)

[https://www.w3cplus.com/css/css-font-metrics-line-height-and-vertical-align.html](https://www.w3cplus.com/css/css-font-metrics-line-height-and-vertical-align.html)

[https://fontforge.github.io/en-US/](https://fontforge.github.io/en-US/)

[https://brunildo.org/test/aspect-lh-table2.html](https://brunildo.org/test/aspect-lh-table2.html)

