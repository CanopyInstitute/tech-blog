# 关于session设置过期的问题

## session过期的相关设置
* session.use_cookies：默认值是1，代表使用cookie传递SessionID。
* session.cookie_lifetime：默认值是0，代表SessionID在客户端Cookie储存的时间，代表浏览器一关闭SessionID就失效。
* session.gc_maxlifetime: 表示session在服务器存储的时间。

## PHP如何销过期的session
当发生请求时，php会根据session.gc_probability/session.gc_divisor的值来决定是否启动GC(扫描所有的session的信息，用当前时间减去session最后修改时间，跟session的最大生存时间比较，超过设置，删除session)。

## 为什么会存在session设置过期时间无效的问题
session会以文件的形式保存在系统目录中，当服务器有多个网站应用时，各自的session文件都会保存在同一个文件目录中，但是GC工作机制不会去区分不同的站点的session。

## 解决办法
* 不同站点设置不同的session存储路径
* 自己设置session存储的开始时间，读取session时计算和当前时间的差值，判断是否过期，过期就销毁session文件