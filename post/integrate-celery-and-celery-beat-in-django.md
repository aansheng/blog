# 在Django中集成Celery与Celery-Beat

[Celery](http://www.celeryproject.org/)是一个基于Python开发的分布式任务队列，而`Celery-Beat`则是一个定时任务，类似于Linux平台下的`crontab`。

# 环境准备

我在`macOS`下面操作的，使用到软件版本如下：

```
pyenv 1.2.11
Python 3.7.3
pip 19.1.1
macOS 10.14.5 (18F132)
```

pip包信息如下

```bash
$ pip freeze
......
amqp==2.4.2
celery==4.3.0
Django==2.2.1
kombu==4.5.0
redis==3.2.1
```

- 创建虚拟环境

```bash
$ pyenv virtualenv 3.7.3 venv
$ pyenv activate venv
```

- 安装依赖包

```bash
$ pip install -U Django Celery
$ pip install -U "celery[redis]"  # 使用redis作为Broker
```

- 运行redis服务

`Broker`用的是`redis`，所以需要启动一个`redis`容器

```bash
$ docker run --name redis -d -p 127.0.0.1:6379:6379 redis
```

> 为什么不用rabbitmq？因为一直安装不上librabbitmq......

# 实践

创建Django的项目

```bash
$ cd ~/Workspace
$ django-admin startproject ansheng
$ cd ansheng
$ tree ./
./
├── ansheng
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 5 files
```

## 配置Celery

添加Celery的配置文件

```bash
$ vim ansheng/celery.py
from __future__ import absolute_import, unicode_literals

import os

from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'ansheng.settings')

app = Celery('ansheng')

# 添加celery的配置，意思就是在Django的settings中寻找以CELERY开头的配置项
app.config_from_object('django.conf:settings', silent=True, namespace='CELERY')

app.autodiscover_tasks()  # 自动注册task，就是会去INSTALLED_APPS中的每个app目录下寻找tasks文件，然后加载里面的任务
```

在`settings`中指定`BROKER`

```bash
$ vim ansheng/settings.py
......
# CELERY CONFIG
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
```

## 创建APP

创建两个app，分别名为`app1`和`app2`

```bash
$ python manage.py startapp app1
$ python manage.py startapp app2
```

将APP注册到`INSTALLED_APPS`中

```bash
$ vim ansheng/settings.py
INSTALLED_APPS = [
    ......
    'app1',
    'app2'
]
```

## 创建任务

分别在两个app中创建celery任务，两个任务都比较简单，单纯的返回`app1 task.`和`app2 task.`

```bash
$ vim app1/tasks.py
from __future__ import absolute_import, unicode_literals

from ansheng.celery import app


@app.task
def app1():
    return "app1 task."

$ vim app2/tasks.py
from __future__ import absolute_import, unicode_literals

from ansheng.celery import app


@app.task
def app2():
    return "app2 task."
```

## 启动celery worker

```bash
$ celery -A ansheng worker -l info


celery@ShengdeMacBook-Pro.local v4.3.0 (rhubarb)

Darwin-18.6.0-x86_64-i386-64bit 2019-05-23 03:49:06

[config]
.> app:         ansheng:0x10bb29828
.> transport:   redis://127.0.0.1:6379/0
.> results:     disabled://
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . app1.tasks.app1
  . app2.tasks.app2

[2019-05-23 03:49:07,236: INFO/MainProcess] Connected to redis://127.0.0.1:6379/0
[2019-05-23 03:49:07,247: INFO/MainProcess] mingle: searching for neighbors
[2019-05-23 03:49:08,278: INFO/MainProcess] mingle: all alone
[2019-05-23 03:49:08,302: INFO/MainProcess] celery@ShengdeMacBook-Pro.local ready.
```

启动之后可以看到上面的`tasks`哪里会加载`app1`和`app2`这两个任务。

## 测试celery异步任务

进入django的shell运行任务测试

```bash
$ python manage.py shell

In [1]: from app1.tasks import app1

In [2]: from app2.tasks import app2
# 同步执行
In [3]: app1()
Out[3]: 'app1 task.'
# 同步执行
In [4]: app2()
Out[4]: 'app2 task.'
# 异步执行，任务执行时会返回一个AsyncResult对象，其中070c3c2a-e4f5-4fff-956f-71f10d08f18f就是这个任务的ID
In [5]: app1.delay()
Out[5]: <AsyncResult: 070c3c2a-e4f5-4fff-956f-71f10d08f18f>
# 异步执行
In [6]: app2.delay()
Out[6]: <AsyncResult: 154a77f2-082f-4e3a-99ae-f394fc8e014b>
```

异步任务执行完毕之后，celery worker会输出任务执行记录

```
[2019-05-23 03:50:14,939: INFO/MainProcess] Received task: app1.tasks.app1[070c3c2a-e4f5-4fff-956f-71f10d08f18f]
[2019-05-23 03:50:14,948: INFO/ForkPoolWorker-2] Task app1.tasks.app1[070c3c2a-e4f5-4fff-956f-71f10d08f18f] succeeded in 0.0013050650000110409s: 'app1 task.'
[2019-05-23 03:50:17,475: INFO/MainProcess] Received task: app2.tasks.app2[154a77f2-082f-4e3a-99ae-f394fc8e014b]
[2019-05-23 03:50:17,479: INFO/ForkPoolWorker-1] Task app2.tasks.app2[154a77f2-082f-4e3a-99ae-f394fc8e014b] succeeded in 0.0011451740000012478s: 'app2 task.'
```

## 定时任务配置

在`settings`中添加`CELERY_BEAT_SCHEDULE`的配置

```bash
$ vim ansheng/settings.py
......
from celery.schedules import crontab
from datetime import timedelta

CELERY_BEAT_SCHEDULE = {
    'task1': {
        'task': 'app1.tasks.app1',
        'schedule': crontab(minute='*/5')  # 每五分钟执行一次
    },
    'task2': {
        'task': 'app2.tasks.app2',
        'schedule': timedelta(seconds=5)  # 每5秒执行一次
    },
}
```

- 启动Celery-Beat

```bash
$ celery -A ansheng beat -l info
celery beat v4.3.0 (rhubarb) is starting.
__    -    ... __   -        _
LocalTime -> 2019-05-23 04:00:07
Configuration ->
    . broker -> redis://127.0.0.1:6379/0
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2019-05-23 04:00:07,078: INFO/MainProcess] beat: Starting...
[2019-05-23 04:00:12,120: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
[2019-05-23 04:00:17,107: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
[2019-05-23 04:00:22,107: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
[2019-05-23 04:00:27,107: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
[2019-05-23 04:00:32,107: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
[2019-05-23 04:00:37,108: INFO/MainProcess] Scheduler: Sending due task task2 (app2.tasks.app2)
```

可以看到任务`task2`是每5秒钟就会执行`app2.tasks.app2`，也可以从celery workr中看到对应的输出

```
[2019-05-23 04:00:12,144: INFO/MainProcess] Received task: app2.tasks.app2[4de6fa28-18a3-436a-9c6f-73512fffefd4]
[2019-05-23 04:00:12,167: INFO/ForkPoolWorker-2] Task app2.tasks.app2[4de6fa28-18a3-436a-9c6f-73512fffefd4] succeeded in 0.00619578399994225s: 'app2 task.'
[2019-05-23 04:00:17,112: INFO/MainProcess] Received task: app2.tasks.app2[8ce96999-23b1-4fa0-a29f-02130e60187e]
[2019-05-23 04:00:17,124: INFO/ForkPoolWorker-1] Task app2.tasks.app2[8ce96999-23b1-4fa0-a29f-02130e60187e] succeeded in 0.004253257999948801s: 'app2 task.'
[2019-05-23 04:00:22,115: INFO/MainProcess] Received task: app2.tasks.app2[4f0a0f47-67f3-4b85-93db-9d1547c2d584]
[2019-05-23 04:00:22,116: INFO/ForkPoolWorker-2] Task app2.tasks.app2[4f0a0f47-67f3-4b85-93db-9d1547c2d584] succeeded in 0.00014236899994557461s: 'app2 task.'
[2019-05-23 04:00:27,113: INFO/MainProcess] Received task: app2.tasks.app2[70d2ab5f-c60d-4828-a789-3f1f00d7711e]
[2019-05-23 04:00:27,114: INFO/ForkPoolWorker-1] Task app2.tasks.app2[70d2ab5f-c60d-4828-a789-3f1f00d7711e] succeeded in 0.00020057100005033135s: 'app2 task.'
[2019-05-23 04:00:32,114: INFO/MainProcess] Received task: app2.tasks.app2[635607a6-803b-4332-b5a4-a65fea89b7ac]
[2019-05-23 04:00:32,115: INFO/ForkPoolWorker-2] Task app2.tasks.app2[635607a6-803b-4332-b5a4-a65fea89b7ac] succeeded in 0.00013156800002889213s: 'app2 task.'
[2019-05-23 04:00:37,112: INFO/MainProcess] Received task: app2.tasks.app2[9a0e016b-aec4-4e4a-b102-1c0e8b9c4e45]
[2019-05-23 04:00:37,113: INFO/ForkPoolWorker-1] Task app2.tasks.app2[9a0e016b-aec4-4e4a-b102-1c0e8b9c4e45] succeeded in 0.0001392950000536075s: 'app2 task.'
```