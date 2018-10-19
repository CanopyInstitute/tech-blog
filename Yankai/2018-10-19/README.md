# Javascript的一个设计缺陷

## 例子
首先看一段很简单的代码
```js
function demo() {
  return {
    hello: 'world'
  };
}
```
很直观，当我们执行 `demo()` 时会返回 `{hello: 'world'}` 。
但是我们这么写

```js
function demo() {
  return 
  {
    hello: 'world'
  };
}
```

这时候执行 `demo()` 时就会返回 `undefined` ，这是由于Javascript有一个自动修复机制——在程序可能有缺陷的时候，自动插入分号补全。所以上面这段代码就被Javascript补全为
```js
function demo() {
  return;
  {
    hello: 'world'
  };
}
```
并且这种错误不会有任何提示。

## 自动分号补齐 - ASI

Javascript实现的词法文法中有一个自动分号补齐（Automatic Semicolon Insertion，简称 ASI）的机制，它的规则为：
> 一些JavaScript语句必须用分号结束，所以会被自动分号补全 (ASI)影响

### ASI的一种机制
在ASI的机制中有一类语句叫做 `restricted productions`（不知道中文叫啥）。简单来说，就是组成这类语句的两个token当中不允许出现换行符 \n。如果在第一个token后面遇到了换行符，则判断语句结束，插入分号。restricted productions包括：

* return xxx （xxx是要返回的对象）
* throw xxx（xxx是要抛出的错误对象）
* break / continue xxx（xxx是循环的标签）
* 作为后缀的 ++ / --

最后一条的原因是需要区分以下的情况

```js
a
++
b
```

因为++/--既可以作为前缀又可以作为后缀，在这样的情况下作为后缀解析会遇上换行符，所以只能作为b的前缀，自动插入分号后变成：

```js
a; ++b;
```

### ASI的另一种机制

当下一行的开头是 `(`, `[`, `/` 这三个字符之一的时候：

```js
a = b
(function () { ... }())
```

就会被解析成：

```js
a = b(function () {...}());
```

换言之，( ) 会被看做是在调用函数b。同理，[ ] 会被看做是在获取b的属性，而被斜杠坑的情况则要求更苛刻一些：

```js
a = b
/Error/i.test(str) && doSomething()
```

第二行的写法本身比较少见，但也不是不可能。结果会被解析成：

```js
a = b / Error / i.test(str) && doSomething();
```

斜杠被解析成了除号！

要躲开这个坑，其实真的挺简单，只要尽量别用这三个字符作为一行开头就行了。事实上遇到比较多的也只有括号开头。真的避不开的话，可以在行头手动加个分号：

```js
a = b
;(function () { ... }())
```

### 额外一点

在 `for` 循环声明里的分号是永远不能省的，ASI不会在 `for` 循环中插分号。

## 最后

现在有许多Javascript编码规范提倡省去分号使代码更简洁精炼，这时候就要留神以上的两个小坑了。

## 参考链接
[http://www.bradoncode.com/blog/2015/08/26/javascript-semi-colon-insertion/](http://www.bradoncode.com/blog/2015/08/26/javascript-semi-colon-insertion/)

[http://www.ecma-international.org/ecma-262/5.1/](http://www.ecma-international.org/ecma-262/5.1/)

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Lexical_grammar#%E8%87%AA%E5%8A%A8%E5%88%86%E5%8F%B7%E8%A1%A5%E5%85%A8](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Lexical_grammar#%E8%87%AA%E5%8A%A8%E5%88%86%E5%8F%B7%E8%A1%A5%E5%85%A8)

[https://www.zhihu.com/question/21076930/answer/17135846](https://www.zhihu.com/question/21076930/answer/17135846)