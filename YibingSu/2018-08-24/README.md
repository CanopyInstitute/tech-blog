# 小程序request请求session失败的问题

### 前段时间做小程序发现一个问题，因为微信的request请求是类似于ajax的，不是浏览器发出的请求，因此请求不会带sessionid，所以获取不到存储在服务器端的session###

## sessionid的作用
sessionid是用来识别用户会话的唯一标识，当浏览器第一次请求服务器时，服务器会生成一个sessionid,并通过response headers中的set-cookie返回给浏览器，浏览器将sessionid存储在cookie中![如图](http://47.98.193.94/tp/Uploads/set-cookie.jpg)

当用户请求时，会在header中附带sessionid的信息，服务器就会将cookie中存储的sessionid和服务器中的sessionid相比较，从而找到用户的session信息。

![img](http://47.98.193.94/tp/Uploads/cookie-1.jpg)

## session和cookie的区别
session和cookie是一种会话机制，sesson是服务器端的机制，cookie是浏览器的机制，session的实现是依赖于cookie的。

## 解决思路
当用户发出请求时，服务器可以返回给前端sessionid，调用wx.request时，手动添加sessioid信息，这样就可以读取存储在服务器的session信。

```
    wx.request({
      url: "http://www.test.com/test1",
      header:{
        'Cookie': 'PHPSESSID=' +'cd4lpqkniof7qijec5muh04hs1'
      },
      data: {},
      success: function (res) {
        console.log(res.data)
      }
    })
```





