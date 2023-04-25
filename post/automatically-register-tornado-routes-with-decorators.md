# 使用装饰器自动注册Tornado路由

## 版本1

在这个版本中，首先创建了`RouterConfig`对象，其构造方法创建了`tornado.web.Application()`并赋值为`self.Application`，在每个`Handler`上添加`@app.route`装饰器，对应的就是`RouterConfig`下面的`route`对象，其中`Handler`实例会被赋值到`handler`参数中，最后把`URL`和`Handler`对应关系添加到路由表中，`URL`在每个`Handler`中创建的属性。

```python
#!/usr/bin/env python
# _*_ coding:utf-8 _*_

# Created by 安生 on 2017/2/9

import tornado
import tornado.web
import tornado.ioloop

class RouterConfig:
    def __init__(self):
        self.Application = tornado.web.Application()  # 创建路由对象

    def route(self, handler):
        self.Application.add_handlers('.*$', [(handler.URL, handler)])  # 路有关系映射添加到路由表中

app = RouterConfig()  # 创建路由

@app.route
class MainHandler(tornado.web.RequestHandler):
    URL = r'/'

    def get(self, *args, **kwargs):
        self.write('Hello, 安生')

@app.route
class MainHandler(tornado.web.RequestHandler):
    URL = r'/hi'

    def get(self, *args, **kwargs):
        self.write('hi, 安生')

if __name__ == "__main__":
    app.Application.listen(8000)
    print("http://127.0.0.1:8000/")
    tornado.ioloop.IOLoop.instance().start()
```

## 版本2

创建`Route`对象，然后再`Handler`上加上装饰器`@route(r'/')`，并把`URL`传递进来，其中对应到`__call__`方法中的`url`参数，然后把路由对应关系以元祖的方式添加到列表中，待所有的路由都添加完成之后，创建Tornado的路有对象，然后把路由表放进去，最后完成注册。

```python
#!/usr/bin/env python
# _*_ coding:utf-8 _*_

# Created by 安生 on 2017/2/9

import tornado.ioloop
import tornado.web

class Route(object):
    """ 把每个URL与Handler的关系保存到一个元组中，然后追加到列表内，列表内包含了所有的Handler """

    def __init__(self):
        self.urls = list()  # 路由列表

    def __call__(self, url, *args, **kwargs):
        def register(cls):
            self.urls.append((url, cls))  # 把路由的对应关系表添加到路由列表中
            return cls

        return register

route = Route()  # 创建路由表对象

@route(r'/')
class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.write('Hello, 安生')

@route(r'/hi')
class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.write('hi, 安生')

application = tornado.web.Application(route.urls)  # 创建app，并且把路有关系放入到Application对象中

if __name__ == '__main__':
    application.listen(8000)
    print("http://127.0.0.1:%s/" % 8000)
    tornado.ioloop.IOLoop.instance().start()
```

## 版本3

这个版本也是我现在在使用版本，原理都一样，这里的特点就是继承Tornado路由对象

```python
#!/usr/bin/env python
# _*_ coding:utf-8 _*_

# Created by 安生 on 2017/2/9

import tornado.web
import tornado.ioloop

class RouterConfig(tornado.web.Application):
    """ 重置Tornado自带的路有对象 """

    def route(self, url):
        """
        :param url: URL地址
        :return: 注册路由关系对应表的装饰器
        """

        def register(handler):
            """
            :param handler: URL对应的Handler
            :return: Handler
            """
            self.add_handlers(".*$", [(url, handler)])  # URL和Handler对应关系添加到路由表中
            return handler

        return register

app = RouterConfig(cookie_secret='ulb7bEIZmwpV545Z')  # 创建Tornado路由对象，默认路由表为空

@app.route(r'/')
class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.write('Hello, 安生')

@app.route(r'/hi')
class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.write('hi, 安生')

if __name__ == "__main__":
    app.listen(8000)
    print("http://127.0.0.1:%s/" % 8000)
    tornado.ioloop.IOLoop.instance().start()
```

## 测试

以上一个版本中，测试方法及输出都是一样的，可以用`requests`模块模拟HTTP请求

```python
>>> import requests
>>> requests.get('http://127.0.0.1:8000/').text
'Hello, 安生'
>>> requests.get('http://127.0.0.1:8000/hi').text
'hi, 安生'
```