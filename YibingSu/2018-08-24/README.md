# С����request����sessionʧ�ܵ�����

### ǰ��ʱ����С������һ�����⣬��Ϊ΢�ŵ�request������������ajax�ģ��������������������������󲻻��sessionid�����Ի�ȡ�����洢�ڷ������˵�session###

## sessionid������
sessionid������ʶ���û��Ự��Ψһ��ʶ�����������һ�����������ʱ��������������һ��sessionid,��ͨ��response headers�е�set-cookie���ظ���������������sessionid�洢��cookie��![��ͼ](http://47.98.193.94/tp/Uploads/set-cookie.jpg)

���û�����ʱ������header�и���sessionid����Ϣ���������ͻὫcookie�д洢��sessionid�ͷ������е�sessionid��Ƚϣ��Ӷ��ҵ��û���session��Ϣ��

![img](http://47.98.193.94/tp/Uploads/cookie-1.jpg)

## session��cookie������
session��cookie��һ�ֻỰ���ƣ�sesson�Ƿ������˵Ļ��ƣ�cookie��������Ļ��ƣ�session��ʵ����������cookie�ġ�

## ���˼·
���û���������ʱ�����������Է��ظ�ǰ��sessionid������wx.requestʱ���ֶ����sessioid��Ϣ�������Ϳ��Զ�ȡ�洢�ڷ�������session�š�

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





