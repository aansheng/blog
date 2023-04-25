# Docker部署Sentry监控Django应用并使用email+钉钉通知

Sentry是一个开源的程序异常跟踪系统，基本上主流的语言，Sentry都支持，Sentry是用`Django+DRF+Celery+Celery-Beat`开发的，如果你是`Pythoner`，并且对这些技术栈都相当熟悉，你可以阅读一下相关的源码，定有不少的收获，这里值得一提的是`Sentry`在部署的时候只支持`Python2`，不支持`Python3`。

这篇文章我们将会以`Docker`的方式部署`Sentry`，这也是官方推荐的部署方式，并且我们会写一个简单的`Django`应用，使用`email+钉钉`的方式进行通知，一旦出现异常，开发人员可以及时收到并处理。

## 环境

我在`vultr`上开了一台云服务器供这次测试使用，用的是`CentOS7`的系统。

### 初始化操作

安装epel源

```bash
$ yum install epel-release -y
```

更新系统

```bash
$ yum update -y
```

安装一些工具包

```bash
$ yum install python-pip vim git -y
```

建议重启下系统

```bash
$ reboot
```

### 基本配置

- 内存

```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           3.7G         95M        3.4G        8.4M        168M        3.4G
Swap:            0B          0B          0B
```

- CPU

```bash
$ cat /proc/cpuinfo | grep processor | wc -l
2
```

配置也就是`2H4G`，如果你是`1G`内存的服务器，貌似我在腾讯云上面跑起来比较艰难，感觉至少还是要`2G`把。

## 安装Docker

CentOS系列的安装文档放在：https://docs.docker.com/install/linux/docker-ce/centos/ ，感兴趣的就可以去阅读，我这里就简化一些操作。

- 安装一些软件包

```bash
$ yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 添加docker的repo源

```bash
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装docker

```bash
$ yum install docker-ce -y
```

- 启动docker

```bash
$ systemctl start docker
```

- 开机自启动

```bash
$ systemctl enable docker
```

- 查看docker版本

```bash
$ docker --version
Docker version 18.03.1-ce, build 9ee9f40
```

- 运行一个`hello-world`

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

如果你运行之后出现的结果和我一样，那么，OK，docker已经安装完成了.

如果你使用的是国内的服务器，可能在`pull`镜像的时候，异常的慢，所以官方就提供了国内的docker镜像加速，[点我点我](http://docker-cn.com/registry-mirror)，配置好之后一定要记得重启下docker的服务，不然无法加载配置，重启命令如下：

```bash
$ systemctl restart docker
```

## 安装docker-compose

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

我装的是`1.21.2`版本，通常你安装的`docker`和`docker-compose`是最新的版本都不会有什么问题，查看一下`docker-compose`版本

```bash
$ docker-compose --version
docker-compose version 1.21.2, build a133471
```

## 安装Sentry

终于到了关键的步骤，`Sentry`，官方提供了一整套`docker`的部署方式来运行`Sentry`。

- 下载项目

```bash
$ cd /opt/
$ git clone https://github.com/getsentry/onpremise.git sentry
$ cd sentry
```

- 创建数据库相关

```bash
$ docker volume create --name=sentry-data && docker volume create --name=sentry-postgres
```

- cp配置文件

```bash
$ cp -n .env.example .env
```

- 添加一些依赖库

```bash
$ vim requirements.txt
# Add plugins here
sentry-dingding~=0.0.2  # 钉钉通知插件 
django-smtp-ssl~=1.0  # 发邮件支持SSL协议
redis-py-cluster==1.3.4
```

- 构建Docker镜像

```bash
$ docker-compose build
```

- 生成密钥

```bash
$ docker-compose run --rm web config generate-secret-key
......
# 最后一行会输出类似如下的秘钥串
kbjodp(id&b0^kbnxijn11&2e6xu&vy1(oini!-zl)pl610n&v
```

把上面的秘钥添加到`.env`文件的`SENTRY_SECRET_KEY`环境变量中

```bash
$ vim .env
# Run `docker-compose run web config generate-secret-key`
# to get the SENTRY_SECRET_KEY value.
SENTRY_SECRET_KEY='kbjodp(id&b0^kbnxijn11&2e6xu&vy1(oini!-zl)pl610n&v'
```

- 生成数据库表并创建管理员用户

```bash
$ docker-compose run --rm web upgrade
......
Would you like to create a user account now? [Y/n]: y  # 创建用户
Email: ianshengme@gmail.com  # 邮箱
Password:  # 密码
Repeat for confirmation:  # 确认密码
Should this user be a superuser? [y/N]: y  # 为超级管理员用户
```

- 启动服务

```bash
$ docker-compose up -d
```

可以通过`docker-compose ps`查看一下启动了那些容器

```bash
$ docker-compose ps
       Name                     Command               State           Ports
------------------------------------------------------------------------------------
sentry_cron_1        /entrypoint.sh run cron          Up      9000/tcp
sentry_memcached_1   docker-entrypoint.sh memcached   Up      11211/tcp
sentry_postgres_1    docker-entrypoint.sh postgres    Up      5432/tcp
sentry_redis_1       docker-entrypoint.sh redis ...   Up      6379/tcp
sentry_smtp_1        docker-entrypoint.sh tini  ...   Up      25/tcp
sentry_web_1         /entrypoint.sh run web           Up      0.0.0.0:9000->9000/tcp
sentry_worker_1      /entrypoint.sh run worker        Up      9000/tcp
```

上面的几个服务，介绍如下：

|名称|描述|
|:--|:--|
|`sentry_cron`|定时任务，使用的是`celery-beat`|
|`sentry_memcached`|memcached|
|`sentry_postgres`|pgsql数据库|
|`sentry_redis`|运行celery需要的服务|
|`sentry_smtp`|发邮件|
|`sentry_web`|使用`django+drf`写的一套`Sentry Web界面`|
|`sentry_worker`|celery的worker服务，用来跑异步任务的|

服务启动之后，默认会监听`9000`端口，如果你想更改，可以在`docker-compose.yml`中进行更改。

我将我的域名`ansheng.me`添加了一条`A`记录，记录值是`sentry`，指向这台sentry的云服务器，所以可以通过`sentry.ansheng.me`进行访问。

## Sentry的基本设置

浏览器打开`http://sentry.ansheng.me:9000/`，输入邮箱和密码进行登录

![1531814821978851](/images/2018/07/1531814821978851.png)

登录成功之后，输入对应的`Root URL`和`Admin Email`，然后点击`Continue`

![1531814926463487](/images/2018/07/1531814926463487.png)

点击之后就进入了`Sentry`的主界面

![1531815010037154](/images/2018/07/1531815010037154.png)

### 添加一个Python项目

点击右上角的`Add new`，选择`Project`

![1531815044620126](/images/2018/07/1531815044620126.png)

然后创建项目

![1531815081686595](/images/2018/07/1531815081686595.png)

项目创建完成之后，会出现一个使用的界面

![153181512463147](/images/2018/07/153181512463147.png)

#### 测试Python程序

按照上面的步骤 ，我们来一步一步的操作，我在我这台服务器上面操作，Python版本如下：

```bash
$ python -V
Python 2.7.5
```

- 安装raven

```bash
$ pip install raven --upgrade
```

- 添加测试代码

```bash
$ vim sentry_python_test.py
from raven import Client

client = Client('http://ce5502f746a4484f9b2c391a54d2d1c4:bca94f6b330a4dd183e68243b2a1b99c@sentry.ansheng.me:9000/2')

try:
    1 / 0
except ZeroDivisionError:
    client.captureException()
```

- 运行

```bash
$ python sentry_python_test.py
Sentry is attempting to send 1 pending error messages
Waiting up to 10 seconds
Press Ctrl-C to quit
```

#### 查看异常

在上面的使用界面中，点击`Got it~ Take me to the Issue Stream.`进入到项目的`Issue`页面，也就是错误页面

![1531815155343037](/images/2018/07/1531815155343037.png)

可以看到已经有一条异常了，这个异常就是我们刚测时捕获的报错，点击`ZeroDivisionError`大标题，进入异常的详细页面。

在项目页面中我们可以看到`MESSAGE`和`EXCEPTION`这两块，输出了具体的错误详情

![153181519029878](/images/2018/07/153181519029878.png)

基本上的步骤就像上面一样，流程都差不多

## 添加Django项目并进行监控

根据上面创建`Python`Sentry项目的步骤添加一个名为`blog`，是`Django`框架的项目，并把`DSN`的记录值保存下来

> http://9ad8168873a94fb1927e14111b9bca1e:29ea550781e24ca2ba0c0ee4829dfc96@sentry.ansheng.me:9000/3

![15318152279307442](/images/2018/07/15318152279307442.png)

### 创建django项目

django的项目我在我的电脑上创建.

- 添加一个名为`blog`的虚拟环境

```bash
$ pyenv virtualenv 3.6.5 blog
```

- 切换虚拟环境

```bash
$ pyenv activate blog
```

- 安装django

```bash
$ pip install django
```

- 创建project

```bash
$ cd /tmp
$ django-admin startproject blog
$ cd blog
$ python manage.py migrate
```

- 启动项目

```
$ python manage.py runserver 0:9999
Performing system checks...

System check identified no issues (0 silenced).
July 17, 2018 - 03:35:05
Django version 2.0.7, using settings 'blog.settings'
Starting development server at http://0:9999/
Quit the server with CONTROL-C.
```

端口监听在`9999`，可以通过curl进行访问测试

```bash
$ curl -I http://127.0.0.1:9999
HTTP/1.1 200 OK
......
```

状态是200表示运行没问题

- 查看目录结构

```bash
➜  blog tree ./
./
├── blog
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── views.py
│   └── wsgi.py
├── db.sqlite3
└── manage.py
```

- 编写一个简单的views

```bash
$ vim blog/views.py
from django.http import HttpResponse


def success(request):
    return HttpResponse("OK")

def error(request):
    1 / 0
    return HttpResponse("Not OK")
```

- 添加url

```bash
$ vim blog/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path('success', views.success, name='success'),
    path('error', views.error, name='error'),
]
```

- 访问测试

访问可以返回成功的API

```bash
$ curl http://127.0.0.1:9999/success
OK
```

访问会报错的API

```bash
$ curl -I http://127.0.0.1:9999/error
HTTP/1.1 500 Internal Server Error
......
```

报了一个`500`的错误，日志输出如下

```
Internal Server Error: /error
Traceback (most recent call last):
  File "/Users/shengan/.pyenv/versions/blog/lib/python3.6/site-packages/django/core/handlers/exception.py", line 35, in inner
    response = get_response(request)
  File "/Users/shengan/.pyenv/versions/blog/lib/python3.6/site-packages/django/core/handlers/base.py", line 128, in _get_response
    response = self.process_exception_by_middleware(e, request)
  File "/Users/shengan/.pyenv/versions/blog/lib/python3.6/site-packages/django/core/handlers/base.py", line 126, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
  File "/private/tmp/blog/blog/views.py", line 8, in error
    1 / 0
ZeroDivisionError: division by zero
[17/Jul/2018 03:50:55] "HEAD /error HTTP/1.1" 500 60675
```

### 集成Sentry

- 安装raven

```bash
$ pip install raven --upgrade
```

- 在`INSTALLED_APPS`中添加`raven.contrib.django.raven_compat`

```python
INSTALLED_APPS = (
    ......
    'raven.contrib.django.raven_compat',
)
```

- 在setting内添加sentry配置

```bash
import os
import raven

RAVEN_CONFIG = {
    'dsn': 'http://9ad8168873a94fb1927e14111b9bca1e:29ea550781e24ca2ba0c0ee4829dfc96@sentry.ansheng.me:9000/3'  # DSN输入我们刚才记录下来的DSN
}
```

- 配置完成之后，我我们再来进行下测试

访问错误的API

```bash
$ curl -I http://127.0.0.1:9999/error
```

- 异常列表

![15318152560393028](/images/2018/07/15318152560393028.png)

- 异常详情

![15318152873324091](/images/2018/07/15318152873324091.png)

## 配置email通知

我这里使用的是网易邮箱，具体操作如下

- 修改config.yml文件添加`Mail Server`的配置

```bash
$ vim config.yml
mail.backend: 'django_smtp_ssl.SSLEmailBackend'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.163.com'
mail.port: 465
mail.username: 'anshengme@163.com'
mail.password: '14RJg5vzGFWUNKiP'  # 这里的密码，不是登录密码，是"客户端授权密码"
mail.use-tls: false
# The email address to send on behalf of
mail.from: 'anshengme@163.com'
```

- 重新build镜像

```bash
$ docker-compose build
$ docker-compose up -d
```

重启之后可以在`Admin`==>`邮件`下面找到邮箱的配置

![15318153344064908](/images/2018/07/15318153344064908.png)

然后点击下面的`向 ianshengme@gmail.com 发送一份测试邮件`，然后进入你的邮箱，查看是否收到了邮件

![15318153599131148](/images/2018/07/15318153599131148.png)

我这里是可以成功收到发送过来的测试邮件，通常邮箱配置完成之后就可以接收到异常通知了。

- 验证用户的邮箱

找到`Settings=>Account=>Emails`

![15318154003174138](/images/2018/07/15318154003174138.png)

然后点击`Resend verification`，会收到一封验证邮件

![1531815423369428](/images/2018/07/1531815423369428.png)

点击`Confirm`确认邮箱。

- 异常通知测试

然后呢，我们在访问一下错误的API

```bash
$ curl -I http://127.0.0.1:9999/error
HTTP/1.1 500 Internal Server Error
```

- 查看邮箱收到的报警邮件

![1531815451551609](/images/2018/07/1531815451551609.png)

基本上你收到的错误邮件就和上面的差不多，不知道为什么那个头像不显示，不管它，如果你要查看错误详情，点击`View on Sentry`就可以打开异常详情了。

## 配置钉钉通知

我们都知道，邮箱通知的时效性太差，不能够即使传递，所以我们重点介绍下`钉钉机器人`的群通知。

- 创建钉钉群组添加自定义机器人并获取到`access_token`

我这里获取到的`access_token`是`4dadfa77dfc3fe3b34e91237665afbef165e745dd93ec191c48bf4843b1ad63c`

- 配置

找到blog项目的所有集成

![1531815482810024](/images/2018/07/1531815482810024.png)

然后找到`DingDing`这个插件，`启用插件`，然后点击`Configure Plugin`配置插件

![1531815509278548](/images/2018/07/1531815509278548.png)

输入刚刚获取到的`access_token`，点击`Save Changes`以保存更改

![1531815528549936](/images/2018/07/1531815528549936.png)

- 测试异常通知

然后我们访问以下异常的API

```bash
$ curl -I http://127.0.0.1:9999/error
HTTP/1.1 500 Internal Server Error
......
```

此时你的钉钉机器人应该会发送如下的报错

![1531815564221066](/images/2018/07/1531815564221066.png)

点击上面的`href`就可以跳转到错误详情页面，到此，本篇文章也就结束了。