# 通过django-rest-framework中的status模块来学习HTTP状态码

Django下面有一个特别实用的框架，就是`django-rest-framework`，用它你可以写出更符合`restful`标准的` api`，本篇会通读`django-rest-framework`下的`status`模块来学习`http`返回的状态码是如何定义的，如果你正在写`API`相关的接口，本篇文章对你会有一定的帮助，希望你能够通读下去。

在开始之前我们需要安装一下`django-rest-framework`的框架：

```bash
pip install djangorestframework
```

## 状态码范围的定义

`HTTP`状态码分为`1xx`,`2xx`,`3xx`,`4xx`,`5xx`，那么每个范围代表什么意思呢？如下表，我已经把它列出来了

|范围|描述|
|:--|:--|
|`1xx`|信息|
|`2xx`|成功|
|`3xx`|重定向|
|`4xx`|客户端错误|
|`5xx`|服务端错误|

然而，在`status`模块中，为我们提供了几个方法用来验证此状态码是否是这个错误，比如说，我使用`requests`模块请求了某个网站，然后我要判断是否请求成功，因为要根据不同的结果做不同的事情，实例代码如下：

```python
from rest_framework import status
import requests

# 定义一个URL
URL = 'https://blog.ansheng.me'

# 发送一次请求
result = requests.get(URL)

# 如果返回值是成功的状态码就输出
if status.is_success(result.status_code):
    print('请求成功！')

# 在发送一个请求
ret = requests.get(URL + '/hello')

# 如果返回值是客户端错误的就输出
if status.is_client_error(ret.status_code):
    print('客户端出错！')
```

返回结果如下：

```bash
/Library/Frameworks/Python.framework/Versions/3.5/bin/python3.5 /Users/ansheng/MyPythonCode/code.py
请求成功！
客户端出错！
```

- 源码是怎么干的？

如果你要问我是怎么知道状态码范围的定义的，那么我只能告诉你，源码告诉我了，所以我知道了。

```bash
def is_informational(code):
    return code >= 100 and code <= 199


def is_success(code):
    return code >= 200 and code <= 299


def is_redirect(code):
    return code >= 300 and code <= 399


def is_client_error(code):
    return code >= 400 and code <= 499


def is_server_error(code):
    return code >= 500 and code <= 599
```

## 常用的HTTP状态码

发现如果要一个一个的说明每个状态码是干什么使的，会浪费很多的精力去看这些不是那么重要的问题，所以我会把几个常用的状态码列举出来，然后剩下的我会推荐几篇文章，如果你真的感兴趣`(虽然不推荐这么做，浪费时间)`。

|状态码|描述|
|:--|:--|
|200|请求处理成功|
|201|当服务端根据客户端的请求创建了新的资源后会返回该状态吗|
|202|根据客户端的请求修改数据时(如果是一个异步的操作，不能确保次修改一定成功)|
|400|客户端发送过来的参数出现了问题|
|401|用户验证失败，指访问该资源需要身份验证，但客户端提供的Token貌似是错误的|
|403|访问的资源是存在的，但是该资源仅允许指定的用户或者IP访问|
|404|要访问的资源不存在|
|405|如果一个资源仅允许GET方法来进行资源的获取，那么你用POST/PUT/DELETE等方法进行访时，是不允许的|
|500|服务器内部错误|
|502|代理访问后端服务无响应|
|504|代理访问后端服务超时，与502一样|

推荐的一些资源。

1. See RFC 2616 - http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
2. And RFC 6585 - http://tools.ietf.org/html/rfc6585
3. And RFC 4918 - https://tools.ietf.org/html/rfc4918