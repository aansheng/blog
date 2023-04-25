# Celery分布式任务队列

Celery是一个基于Python开发的分布式任务队列，关于Celery的更多介绍可以参考官网。

要使用`Celery`我们需要一个`Broker`和`Backend`，这里我们使用`Redis`。

- GitHub官网：https://github.com/celery/celery
- Docs：http://docs.celeryproject.org/en/latest/
- Broker：接收和发送任务消息
- Backend：存储任务执行结果

> Celery不支持Windows

## 安装

- redis

这里为了方便起见，我们使用`Docker`来快速启动一个`Redis`服务。

```bash
➜  ~ docker pull redis
➜  ~ docker run -d --name redis -p 6379:6379 redis
```

- celery

我已经通过`pyenv`创建好了一个虚拟环境，切换过去即可

```bash
➜  ~ pyenv activate venv
(venv) ➜  ~ pip install celery
```

因为我们的`Broker`和`Backend`使用的都是`Redis`，所以还需要安装`redis`模块

```bash
(venv) ➜  ~ pip install redis
```

## 使用

### 应用

创建一个文件`tasks.py`，用来存放任务列表

```python
from celery import Celery

app = Celery('tasks', broker='redis://localhost',backend='redis://localhost')

@app.task
def add(x, y):
    return x + y
```

`Celery`的第一个参数是这个APP的名字，`broker`和`backend`指定了要使用消息代理的URL。

上面我们定义了一个任务叫`add`，它返回`x`和`y`的和。


### 启动

```bash
(venv) ➜  ~ celery -A tasks worker --loglevel=info
```

执行启动命令的时候，一定要和`tasks.py`在同一级目录下，执行成功的输出应该如下：

```bash
(venv) ➜  ~ celery -A tasks worker --loglevel=info

celery@ShengdeMacBook-Pro.local v4.1.0 (latentcall)

Darwin-17.3.0-x86_64-i386-64bit 2018-01-22 14:33:49

[config]
.> app:         tasks:0x1069cd3c8
.> transport:   redis://localhost:6379//
.> results:     redis://localhost/
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.add

[2018-01-22 14:33:49,334: INFO/MainProcess] Connected to redis://localhost:6379//
[2018-01-22 14:33:49,350: INFO/MainProcess] mingle: searching for neighbors
[2018-01-22 14:33:50,376: INFO/MainProcess] mingle: all alone
[2018-01-22 14:33:50,394: INFO/MainProcess] celery@ShengdeMacBook-Pro.local ready.
```

### 调用任务

新打开一个终端并切换到虚拟环境中，然后进入Python交互模式下

```python
>>> from tasks import add
# 如果执行函数的时候不加delay则表示当做函数来执行
>>> add(1,2)
3
# 异步任务添加成功后返回一个AsyncResult实例
>>> add.delay(1,2)
<AsyncResult: e167ae38-0e53-4592-ad0d-6c685f16168c>
>>> result = add.delay(1,2)
# get获取执行结果，如果任务没有执行完成就执行get则会阻塞住
>>> result.get()
3
```

为了测试延迟异步效果，我们在tasks.py里面新加一个任务叫`sleep`

```python
from celery import Celery
import time

app = Celery('tasks', broker='redis://localhost',backend='redis://localhost')

@app.task
def add(x, y):
    return x + y

@app.task
def sleep(t):
    time.sleep(t)
    return str(time.time())
```

任务添加完成之后我们需要关闭`celery worker`，然后在启动，当我们再次启动的时候回发现刚添加的任务`sleep`已经加入到tasks中了。

```bash
(venv) ➜  ~ celery -A tasks worker --loglevel=info

celery@ShengdeMacBook-Pro.local v4.1.0 (latentcall)

Darwin-17.3.0-x86_64-i386-64bit 2018-01-22 14:34:27

[config]
.> app:         tasks:0x105647400
.> transport:   redis://localhost:6379//
.> results:     redis://localhost/
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.add
  . tasks.sleep

[2018-01-22 14:34:27,548: INFO/MainProcess] Connected to redis://localhost:6379//
[2018-01-22 14:34:27,563: INFO/MainProcess] mingle: searching for neighbors
[2018-01-22 14:34:28,588: INFO/MainProcess] mingle: all alone
[2018-01-22 14:34:28,619: INFO/MainProcess] celery@ShengdeMacBook-Pro.local ready.
```

再次进入python交互环境调用任务

```python
>>> from tasks import sleep
# 创建任务，sleep 10秒
>>> task = sleep.delay(10)
# 在任务还没有执行完成的之后get结果，设置超时时间为1秒，从结果可以看出，任务并没有执行完成，抛出了一个celery.exceptions.TimeoutError的异常
>>> task.get(timeout=1)
Traceback (most recent call last):
  File "/Users/shengan/.pyenv/versions/venv/lib/python3.6/site-packages/celery/backends/async.py", line 256, in _wait_for_pending
    on_interval=on_interval):
  File "/Users/shengan/.pyenv/versions/venv/lib/python3.6/site-packages/celery/backends/async.py", line 55, in drain_events_until
    raise socket.timeout()
socket.timeout

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/shengan/.pyenv/versions/venv/lib/python3.6/site-packages/celery/result.py", line 194, in get
    on_message=on_message,
  File "/Users/shengan/.pyenv/versions/venv/lib/python3.6/site-packages/celery/backends/async.py", line 189, in wait_for_pending
    for _ in self._wait_for_pending(result, **kwargs):
  File "/Users/shengan/.pyenv/versions/venv/lib/python3.6/site-packages/celery/backends/async.py", line 260, in _wait_for_pending
    raise TimeoutError('The operation timed out.')
celery.exceptions.TimeoutError: The operation timed out.
```

如果你要想要知道一个耗时的任务有没有执行完毕，通过上述的测试可以捕获异常，然后做对应的处理，但是celery提供了一个更方便的方法。

```python
>>> result.ready()
False
>>> result.ready()
True
>>> result.get()
'1516603282.2424462'
```

关于AsyncResult的结果对象可以参考：http://docs.celeryproject.org/en/latest/reference/celery.result.html

### 项目中使用celery

```bash
# 项目目录
(venv) ➜  ~ mkdir celery_tasks
# celery的配置文件
(venv) ➜  ~ touch celery_tasks/celery.py
# 任务1
(venv) ➜  ~ touch celery_tasks/task_1.py
# 任务2
(venv) ➜  ~ touch celery_tasks/task_2.py
```

- celery_tasks/celery.py

```python
from __future__ import absolute_import
from celery import Celery

app = Celery('celery_tasks',
             broker='redis://localhost',
             backend='redis://localhost',
             include=['celery_tasks.task_1', 'celery_tasks.task_2'])

app.conf.update(
    result_expires=3600,
)

if __name__ == '__main__':
    app.start()
```

- celery_tasks/task_1.py

```python
from __future__ import absolute_import
from .celery import app


@app.task
def add(x, y):
    return x + y
```

- celery_tasks/task_2.py

```python
from __future__ import absolute_import
import time
from .celery import app


@app.task
def sleep(t):
    time.sleep(t)
    return str(time.time())
```

- 启动

```bash
(venv) ➜  ~ celery -A celery_tasks worker -l info

celery@ShengdeMacBook-Pro.local v4.1.0 (latentcall)

Darwin-17.3.0-x86_64-i386-64bit 2018-01-22 14:59:33

[config]
.> app:         celery_tasks:0x109e29ef0
.> transport:   redis://localhost:6379//
.> results:     redis://localhost/
.> concurrency: 4 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . celery_tasks.task_1.add
  . celery_tasks.task_2.sleep

[2018-01-22 14:59:34,042: INFO/MainProcess] Connected to redis://localhost:6379//
[2018-01-22 14:59:34,072: INFO/MainProcess] mingle: searching for neighbors
[2018-01-22 14:59:35,109: INFO/MainProcess] mingle: all alone
[2018-01-22 14:59:35,148: INFO/MainProcess] celery@ShengdeMacBook-Pro.local ready.
```

新开一个窗口测试

```python
(venv) ➜  ~ python
>>> from celery_tasks import task_1,task_2
>>> task_1.add.delay(1,2)
<AsyncResult: 8621cc57-ddf1-4188-a21d-78678611c735>
>>> task_2.sleep.delay(5)
<AsyncResult: a842caa1-3162-460c-b49a-860797f7784b>
```

### 后台运行

- 启动

```bash
(venv) ➜  ~ celery multi start celery_demo -A celery_tasks -l info
celery multi v4.1.0 (latentcall)
> Starting nodes...
	> celery_demo@ShengdeMacBook-Pro.local: OK
```

- 查看进程

```bash
(venv) ➜  ~ ps -ef | grep celery_demo | grep -v grep
  501 30862     1   0  3:03下午 ??         0:00.72 /Users/shengan/.pyenv/versions/3.6.3/envs/venv/bin/python -m celery worker -A celery_tasks -l info --logfile=celery_demo%I.log --pidfile=celery_demo.pid --hostname=celery_demo@ShengdeMacBook-Pro.local
  501 30900 30862   0  3:03下午 ??         0:00.01 /Users/shengan/.pyenv/versions/3.6.3/envs/venv/bin/python -m celery worker -A celery_tasks -l info --logfile=celery_demo%I.log --pidfile=celery_demo.pid --hostname=celery_demo@ShengdeMacBook-Pro.local
  501 30901 30862   0  3:03下午 ??         0:00.01 /Users/shengan/.pyenv/versions/3.6.3/envs/venv/bin/python -m celery worker -A celery_tasks -l info --logfile=celery_demo%I.log --pidfile=celery_demo.pid --hostname=celery_demo@ShengdeMacBook-Pro.local
  501 30902 30862   0  3:03下午 ??         0:00.01 /Users/shengan/.pyenv/versions/3.6.3/envs/venv/bin/python -m celery worker -A celery_tasks -l info --logfile=celery_demo%I.log --pidfile=celery_demo.pid --hostname=celery_demo@ShengdeMacBook-Pro.local
  501 30903 30862   0  3:03下午 ??         0:00.01 /Users/shengan/.pyenv/versions/3.6.3/envs/venv/bin/python -m celery worker -A celery_tasks -l info --logfile=celery_demo%I.log --pidfile=celery_demo.pid --hostname=celery_demo@ShengdeMacBook-Pro.local
```

一个守护进程，`worker`个数等于`CPU`个数。

- 重启

```bash
(venv) ➜  ~ celery multi restart celery_demo -A celery_tasks -l info
celery multi v4.1.0 (latentcall)
> Stopping nodes...
	> celery_demo@ShengdeMacBook-Pro.local: TERM -> 31132
> Waiting for 1 node -> 31132.....
	> celery_demo@ShengdeMacBook-Pro.local: OK
> Restarting node celery_demo@ShengdeMacBook-Pro.local: OK
> Waiting for 1 node -> None...
```

- 停止

```bash
(venv) ➜  ~ celery multi stop celery_demo -A celery_tasks -l info
celery multi v4.1.0 (latentcall)
> Stopping nodes...
	> celery_demo@ShengdeMacBook-Pro.local: TERM -> 30862
```

- 等待任务执行完毕后停止

```bash
(venv) ➜  ~ celery multi stopwait celery_demo -A celery_tasks -l info
```

## 后记

`Celery`不仅可以做异步任务，还可以做[定时任务](http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html)，它还与`django`做了深度结合，可以使用[django-celery-result](https://github.com/celery/django-celery-results)库将任务执行的结果存放到django的models中，[django-celery-beat](https://github.com/celery/django-celery-beat)库会将定时任务的规则存入到数据库中，而不用通过配置文件来定义。
