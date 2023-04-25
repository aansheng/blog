# Django ORM获取指定天数据的几种方式

## 环境准备

假设你已经创建了虚拟环境并安装好了`Django`

- 查看版本

```bash
# 系统是Ubuntu
$ uname -a
Linux Ubuntu 4.10.0-19-generic #21-Ubuntu SMP Thu Apr 6 17:04:57 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
# 我用的是django 1.11.2版的
$ django-admin --version
1.11.2
```

- 创建项目

```bash
$ django-admin startproject d
```

- 创建app

```bash
$ python manage.py startapp db
```

- 创建models

```python
# - db/models.py
from django.db import models


# Create your models here.

class Language(models.Model):
    name = models.CharField(max_length=15)
    add_time = models.DateTimeField()
```

- 在`settings.py`注册`app`

```python
INSTALLED_APPS = [
    ...
    'db',
]
```

- 初始化数据库

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

- 添加测试数据

```python
# python manage.py shell
from db.models import Language
from django.utils import timezone
from datetime import timedelta,date
current_date=timezone.now()  # 当前时间 2017-06-04 19:48:30
Language.objects.create(name='Python',add_time=current_date - timedelta(days=1))
Language.objects.create(name='C',add_time=current_date - timedelta(days=1))
Language.objects.create(name='C++',add_time=current_date - timedelta(days=2))
Language.objects.create(name='Java',add_time=current_date - timedelta(days=2))
Language.objects.create(name='Go',add_time=current_date - timedelta(days=3))
Language.objects.create(name='JS',add_time=current_date - timedelta(days=3))
```

## 实操

以下的操作都是为了获取`2017-06-03`的数据，过滤出来的结果应该都是下面的内容

```python
<QuerySet [{'id': 1, 'add_time': datetime.datetime(2017, 6, 3, 11, 17, 31, 848500, tzinfo=<UTC>), 'name': 'Python'}, {'id': 2, 'add_time': datetime.datetime(2017,
6, 3, 11, 17, 31, 848500, tzinfo=<UTC>), 'name': 'C'}]>
```

- contains

过滤年月日字符串是否存在

```python
Language.objects.filter(add_time__contains=date(2017, 6, 3)).values()
```

- startswith


是否以指定年月日开头

```python
Language.objects.filter(add_time__startswith=date(2017, 6, 3)).values()
```

- `__year/__month/__day`

制定年月日

```python
Language.objects.filter(add_time__year=2017,add_time__month=6,add_time__day=3).values()
```

- date

`date`是在`django1.9.x`新增的

```python
Language.objects.filter(add_time__date=date(2017, 6, 3)).values()
```

- range

制定一个日期的范围

```python
Language.objects.filter(add_time__range=(current_date, current_date + timedelta(days=1))).values()
```

在`stackoverflow`也有一篇比较好的回答： 

* [How can I filter a date of a DateTimeField in Django?](https://stackoverflow.com/questions/1317714/how-can-i-filter-a-date-of-a-datetimefield-in-django)

> When USE_TZ is True, fields are converted to the current time zone before filtering.
