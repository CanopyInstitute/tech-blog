# 那些年，我们踩过的PHP的坑
>作者 许怀远

>来自 [https://zhuanlan.zhihu.com/p/28490854](https://zhuanlan.zhihu.com/p/28490854)

## 弱类型

==和===异同这种太过低级的坑就直接跳过了，先看一个稍微隐蔽点的坑

```
function translate($keyword)
{
    $trMap = [
        'baidu' => '百度',
        'sougou' => '搜狗',
        '360' => '360',
        'google' => '谷歌'
    ];
    foreach ($trMap as $key => $value) {
        if (strpos($keyword, $key) !== false) {
            return $value;
        }
    }
    return '其他';
}

echo translate("baidu") . "\n";
echo translate("360") . "\n";

```
期待结果
```
百度
360

```
实际结果
```
百度
其他
```
仔细检查，没有string和int的混用，比较也都是用的 !== ，没有用==，为什么还会掉坑里？
问题出在了array上面，虽然你写的是
```
$trMap = [
        'baidu' => '百度',
        'sougou' => '搜狗',
        '360' => '360',
        'google' => '谷歌'
    ];
```
但是PHP给你处理成了
```
array(4) {
  ["baidu"]=>
  string(6) "百度"
  ["sougou"]=>
  string(6) "搜狗"
  [360]=>
  string(3) "360"
  ["google"]=>
  string(6) "谷歌"
}
```
360变成了int类型，这个时候strpos不该报错吗？不，当然是原谅它啦，它选择兼容int
>If needle is not a string, it is converted to an integer and applied as the ordinal value of a character

那么正确的写法是怎么样的呢？稍加改动即可
```
strpos($keyword, $key) //改为 strpos($keyword, (string) $key)
```
可怕之处在于

* 自以为用了===就安全了，忽视了弱类型无处不在这个隐患
* 你可能并没有仔细看每一个函数的说明，没有逐个核对每个参数的类型
* 引发的bug不一定能重现，也有可能平时不会触发，但是留下了安全漏洞

如何100%的避免弱类型的坑？答案是换强类型语言。如果不能换呢？通过以下准则，虽然做不到100%避免，但是做到99.99%是有希望的。

1.能用===/!==的地方，绝不用==/!=，知道类型的情况下，先强转再用===比较
2.调用函数的时候，如果你知道参数类型，在调用时强制转换一下，不能嫌麻烦
我说的是弱类型，不是动态类型，两者不是一码事，不要误会。Python是动态类型强类型，PHP是动态类型弱类型，C语言是静态类型弱类型。如果可以选择，我宁可PHP放弃弱类型，因为弱类型带来的麻烦，已经超出它的便利了。提供一个strict运行模式也行，给足大家十年八年时间慢慢迁移。