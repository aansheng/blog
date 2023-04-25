# Django模型常见问题

## 如何查看queryset的SQL语句

有时你想知道`Django ORM`如何执行我们的查询或者你正在编写的代码的对应SQL是什么，你可以通过`queryset.query`的`str`以获取执行SQL语句

- 示例1

获取Event所有的记录

```python
>>> from events.models import Event
>>> queryset = Event.objects.all()
>>> str(queryset.query)
'SELECT "events_event"."id", "events_event"."epic_id", "events_event"."details", "events_event"."years_ago" FROM "events_event"'
```

- 示例2

获取Event years_ago大于5的记录

```python
>>> queryset = Event.objects.filter(years_ago__gt=5)
>>> str(queryset.query)
'SELECT "events_event"."id", "events_event"."epic_id", "events_event"."details", "events_event"."years_ago" FROM "events_event" WHERE "events_event"."years_ago" > 5'
```

- 实例3

获取非Queryset的query

```python
>>> from django.db import connection
>>> e = Event(epic_id='12', years_ago='2019')
>>> e.save()
>>> print(connection.queries)
```

## 如何在Django ORM中进行OR查询

- 添加测试数据

```python
>>> from django.contrib.auth import get_user_model
>>> User = get_user_model()
>>> User.objects.create_user(username='yash', first_name='Yash', last_name='Rastogi')
<User: yash>
>>> User.objects.create_user(username='John', first_name='John', last_name='Kumar')
<User: John>
>>> User.objects.create_user(username='Ricky', first_name='Ricky', last_name='Dayal')
<User: Ricky>
>>> User.objects.create_user(username='sharukh', first_name='Sharukh', last_name='Misra')
<User: sharukh>
>>> User.objects.create_user(username='Ritesh', first_name='Ritesh', last_name='Deshmukh')
<User: Ritesh>
>>> User.objects.create_user(username='Billy', first_name='Billy', last_name='Sharma')
<User: Billy>
>>> User.objects.create_user(username='Radha', first_name='Radha', last_name='George')
<User: Radha>
>>> User.objects.create_user(username='sohan', first_name='Sohan', last_name='Upadhyay')
<User: sohan>
>>> User.objects.create_user(username='Raghu', first_name='Raghu', last_name='Khan')
<User: Raghu>
>>> User.objects.create_user(username='rishab', first_name='Rishab', last_name='Deol')
<User: rishab>
```

- 需求

假设要查找first_name以P开头或者last_name以D开头的用户

Django提供两种方案:

1. `queryset_1 | queryset_2`
2. `filter(Q(<condition_1>)|Q(<condition_2>)`

- 方案1

```python
>>> queryset = User.objects.filter(first_name__startswith='R') | User.objects.filter(last_name__startswith='D')
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```

- 方案2

通过`Q`进行查询

```python
>>> from django.db.models import Q
>>> qs = User.objects.filter(Q(first_name__startswith='R')|Q(last_name__startswith='D'))
>>> qs
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```

## 如何在Django ORM中进行AND查询

- 需求

假设要查找first_name以P开头并且last_name以D开头的用户

Django提供三种方案:

1. `filter(<condition_1>, <condition_2>)`
2. `queryset_1 & queryset_2`
3. `filter(Q(<condition_1>) & Q(<condition_2>))`

- 方案1

组合多个条件的默认filter方式是AND，所以你可以这样做

```python
>>> queryset_1 = User.objects.filter(first_name__startswith='R', last_name__startswith='D')
>>> queryset_1
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: rishab>]>
```

- 方案2

使用`＆`运算符

```python
>>> queryset_2 = User.objects.filter(first_name__startswith='R') & User.objects.filter(last_name__startswith='D')
>>> queryset_2
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: rishab>]>
```

- 方案3

使用`Q`对象

```python
>>> queryset_3 = User.objects.filter(Q(first_name__startswith='R') & Q(last_name__startswith='D'))
>>> queryset_3
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: rishab>]>
```

- 查看生成查询的SQL语句是否完全相等

```ipython
>>> str(queryset_1.query) == str(queryset_2.query) == str(queryset_3.query)
True
```

## 如何在Django queryset中进行NOT查询

- 需求

使用NOT操作获取 id < 5的所有用户

Django提供两种方案:

1. `exclude(<condition>)`
2. `filter(~Q(<condition>))`

- 方案2

```python
>>> queryset = User.objects.filter(~Q(id__lt=5))
>>> queryset
<QuerySet [<User: Ritesh>, <User: Billy>, <User: Radha>, <User: sohan>, <User: Raghu>, <User: rishab>]>
```

## 如何从相同或不同的Model中联合两个查询集

UNION运算符用于组合两个或多个查询集的结果集，查询集可以来自相同或不同的模型，当它们的查询集来自不同的模型时，字段及其数据类型应该匹配。

使用User模型生成2个查询集来执行联合操作

```python
>>> q1 = User.objects.filter(id__gte=5)
>>> q2 = User.objects.filter(id__lte=9)
>>> q1.union(q2)
<QuerySet [<User: yash>, <User: John>, <User: Ricky>, <User: sharukh>, <User: Ritesh>, <User: Billy>, <User: Radha>, <User: sohan>, <User: Raghu>, <User: rishab>]>
>>> q2.union(q1)
<QuerySet [<User: yash>, <User: John>, <User: Ricky>, <User: sharukh>, <User: Ritesh>, <User: Billy>, <User: Radha>, <User: sohan>, <User: Raghu>, <User: rishab>]>
```

由于`Hero`和`Villain`都有`name`和`gender`，我们可以`values_list`用来限制选定的字段然后做一个联合

```python
Hero.objects.all().values_list(
    "name", "gender"
).union(
Villain.objects.all().values_list(
    "name", "gender"
))
```

这会给你所有`Hero`和`Villain`对象的name和gender。

## 如何仅在查询集中选择某些字段

- 需求

我们想要查询first_name以R开头所有用户的first_name和last_name

Django提供了两种方法来实现这一点：

1. values(输出字典) and values_list(输出列表) methods on queryset.
2. only_method

- 方案1

```ipython
>>> queryset = User.objects.filter(first_name__startswith='R').values('first_name', 'last_name')
>>> queryset
<QuerySet [{'first_name': 'Ricky', 'last_name': 'Dayal'}, {'first_name': 'Ritesh', 'last_name': 'Deshmukh'}, {'first_name': 'Radha', 'last_name': 'George'}, {'first_name': 'Raghu', 'last_name': 'Khan'}, {'first_name': 'Rishab', 'last_name': 'Deol'}]>
```

可以查看生成的SQL语句

```python
>>> str(queryset.query)
'SELECT "auth_user"."first_name", "auth_user"."last_name" FROM "auth_user" WHERE "auth_user"."first_name" LIKE R% ESCAPE \'\\\''
```

- 方案2

```python
>>> queryset = User.objects.filter(first_name__startswith='R').only("first_name", "last_name")
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```

查看生成的SQL语句

```python
>>> str(queryset.query)
'SELECT "auth_user"."id", "auth_user"."first_name", "auth_user"."last_name" FROM "auth_user" WHERE "auth_user"."first_name" LIKE R% ESCAPE \'\\\''
```

方案1和方案2唯一的区别是`only`会输出`id`而方案1却不会。

## 如何在Django中执行子查询表达式

Django允许使用SQL子查询，有一个与User表相关联的UserParent Model。所有的UserParent都有UserParent

```python
>>> from django.db.models import Subquery
>>> users = User.objects.all()
>>> UserParent.objects.filter(user_id__in=Subquery(users.values('id')))
<QuerySet [<UserParent: UserParent object (2)>, <UserParent: UserParent object (5)>, <UserParent: UserParent object (8)>]>
```

## 如何使用基于比较字段值的条件过滤查询集

- 创建测试数据

```python
>>> User.objects.create_user(email="shabda@example.com", username="shabda", first_name="Shabda", last_name="Raaj")
<User: shabda>
>>> User.objects.create_user(email="guido@example.com", username="Guido", first_name="Guido", last_name="Guido")
<User: Guido>
```

- 需求

查询`first_name`和`last_last`相等的用户

```python
>>> from django.db.models import F
>>> User.objects.filter(last_name=F("first_name"))
<QuerySet [<User: Guido>]>
```

```python
>>> User.objects.create_user(email="guido@example.com", username="Tim", first_name="Tim", last_name="Teters")
<User: Tim>
>>> from django.db.models.functions import Substr
>>> User.objects.annotate(first=Substr("first_name", 1, 1), last=Substr("last_name", 1, 1)).filter(first=F("last"))
<QuerySet [<User: Guido>, <User: Tim>]>
```

## 如何在没有任何文件的情况下过滤FileField

`FileField`或`ImageField`存储文件或图像的路径，在DB级别，它们与`CharField`相同。

因此，要找到没有任何文件的FileField，我们可以查询。

```python
no_files_objects = MyModel.objects.filter(
    Q(file='')|Q(file=None)
)
```

## 如何使用Django ORM找到第二条记录

可以使用切片运算符从查询中找到第N条记录

```python
>>> user = User.objects.order_by('-last_login')[1]  # 第二最高记录
>>> user.first_name
'John'
>>> user = User.objects.order_by('-last_login')[2]  # 第三最高记录
>>> user.first_name
'Ricky'
```

如果你想查看到底执行了什么SQL语句，其实使用的就是`LIMIT ... OFFSET`

```sql
SELECT "auth_user"."id",
       "auth_user"."password",
       "auth_user"."last_login",
       "auth_user"."is_superuser",
       "auth_user"."username",
       "auth_user"."first_name",
       "auth_user"."last_name",
       "auth_user"."email",
       "auth_user"."is_staff",
       "auth_user"."is_active",
       "auth_user"."date_joined"
FROM "auth_user"
ORDER BY "auth_user"."last_login" DESC
LIMIT 1
OFFSET 2
```

## 如何查找具有重复字段值的行

- 修改测试数据

```python
>>> User.objects.filter(pk__in=[2,11,13]).update(first_name='John')
3
```

可以使用以下技术找到重复记录

```python
>>> from django.db.models import Count
>>> duplicates = User.objects.values('first_name').annotate(name_count=Count('first_name')).filter(name_count__gt=1)
>>> duplicates
<QuerySet [{'first_name': 'John', 'name_count': 3}]>
```

如果你想到所有的记录，可以这样做

```python
>>> records = User.objects.filter(first_name__in=[item['first_name'] for item in duplicates])
>>> print([item.id for item in records])
[2, 11, 13]
```

## 如何从queryset中查找不同的字段值

您想要查找名称未重复的用户。你可以这样做

```python
>>> distinct = User.objects.values('first_name').annotate(name_count=Count('first_name')).filter(name_count=1)
>>> records = User.objects.filter(first_name__in=[item['first_name'] for item in distinct])
>>> records
<QuerySet [<User: yash>, <User: Ricky>, <User: sharukh>, <User: Ritesh>, <User: Billy>, <User: Radha>, <User: sohan>, <User: Raghu>, <User: rishab>, <User: Guido>]>
>>> [item.pk for item in records]
[1, 3, 4, 5, 6, 7, 8, 9, 10, 12]
```

## 如何使用Q对象进行复杂查询

- OR查询

```python
>>> queryset = User.objects.filter(Q(first_name__startswith='R') | Q(last_name__startswith='D'))
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```

- AND查询

```python
>>> queryset = User.objects.filter(Q(first_name__startswith='R') & Q(last_name__startswith='D'))
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: rishab>]>
```

- 如果要查找first_name以R开头但last_name不是以Z开头

```python
>>> queryset = User.objects.filter(Q(first_name__startswith='R') & ~Q(last_name__startswith='Z'))
>>> queryset
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
>>> [(item.first_name,item.last_name) for item in queryset]
[('Ricky', 'Dayal'), ('Ritesh', 'Deshmukh'), ('Radha', 'George'), ('Raghu', 'Khan'), ('Rishab', 'Deol')]
```

查看生成的SQL语句

```python
>>> str(queryset.query)
'SELECT "auth_user"."id", "auth_user"."password", "auth_user"."last_login", "auth_user"."is_superuser", "auth_user"."username", "auth_user"."first_name", "auth_user"."last_name", "auth_user"."email", "auth_user"."is_staff", "auth_user"."is_active", "auth_user"."date_joined" FROM "auth_user" WHERE ("auth_user"."first_name" LIKE R% ESCAPE \'\\\' AND NOT ("auth_user"."last_name" LIKE Z% ESCAPE \'\\\'))'
```

你可以以更复杂的方式组合Q对象以生成复杂查询

## 如何在Django ORM中对记录进行分组

在Django ORM的记录分组可以使用诸如聚合函数来完成Max，Min，Avg，Sum。

```python
>>> from django.db.models import Avg, Max, Min, Sum, Count
>>> User.objects.all().aggregate(Avg('id'))
{'id__avg': 7.0}
>>> User.objects.all().aggregate(Max('id'))
{'id__max': 13}
>>> User.objects.all().aggregate(Min('id'))
{'id__min': 1}
>>> User.objects.all().aggregate(Sum('id'))
{'id__sum': 91}
```

## 如何从模型中获取随机对象

假设我们有这样的一个model

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
```

- 生成测试数据

我们在Category表中插入一万条记录

```python
>>> from uuid import uuid1
>>> for _ in range(10001):Category.objects.create(name=str(uuid1()))
```

我们想随机获取一个类别，可以通过以下两种方式。

- 方式1

最简单的方法是通过order_by随机获取一条记录

```python
>>> def get_order_by_object():
...     return Category.objects.order_by("?").first()
...
>>> get_order_by_object()
<Category: f81e92be-715e-11e9-a092-8c85906ba0f8>
>>> get_order_by_object()
<Category: f5c0688a-715e-11e9-a092-8c85906ba0f8>
```

> order_by('?')查询可能很昂贵且速度很慢，具体取决于使用的数据库后端

- 方式2

或者另外一种方法，可以获取到最小ID和最大ID，生成范围[min_id，max_id]中的随机数，然后对其进行过滤

```python
>>> from django.db.models import Max, Min
>>> from random import randint
>>> def get_random_object():
...     data = Category.objects.all().aggregate(max_id=Max("pk"), min_id=Min("pk"))
...     while True:
...         pk = randint(data["min_id"], data["max_id"])
...         return Category.objects.filter(pk=pk).first()
...
>>> get_random_object()
<Category: f8b6a8f6-715e-11e9-a092-8c85906ba0f8>
>>> get_random_object()
<Category: facf7c3a-715e-11e9-a092-8c85906ba0f8>
```

## 如何一次创建多个对象

有些时候我们想要一次保存多个对象，假设我们想要一次添加多个用户，我们不希望对数据库进行多次查询。我们可以`bulk_create`用于一次创建多个对象

```python
>>> from django.contrib.auth import get_user_model
>>> User = get_user_model()
>>> User.objects.count()
13
>>> User.objects.bulk_create([User(username='user1'), User(username='user2'), User(username='user3')])
[<User: user1>, <User: user2>, <User: user3>]
>>> User.objects.count()
19
```

## 如何复制或克隆现有模型对象

实例`pk`设置为`None`的情况下保存实例，则会在数据库中创建新记录，这意味着除了`PK`之外的每个字段。

假设有这样一个model

```python
class Article(models.Model):
    title = models.CharField(max_length=255)
    desc = models.CharField(max_length=255)
    content = models.CharField(max_length=255)
```

- 示例

```python
>>> from events.models import Article
>>> Article.objects.count()
0
>>> article = Article.objects.create(title="标题",desc="描述",content="正文")
>>> article.pk
1
>>> Article.objects.count()
1
>>> article.pk = None
>>> article.save()
>>> article.pk
2
>>> Article.objects.count()
2
>>> Article.objects.values()
<QuerySet [{'id': 1, 'title': '标题', 'desc': '描述', 'content': '正文'}, {'id': 2, 'title': '标题', 'desc': '描述', 'content': '正文'}]>
```

## 如何在模型中关联自引用ForeignKey

自引用外键用于建模嵌套关系或递归关系

```python
class Comment(models.Model):
    reply = models.ForeignKey('self', on_delete=models.CASCADE)

# 或者

class Comment(models.Model):
    reply = models.ForeignKey('events.Comment', on_delete=models.CASCADE)
```

## 如何将现有数据库结构转换为Django模型

Django附带了一个名为`inspectdb`的实用程序，它可以通过现有的数据库来创建模型

例如我们在`postgresql`数据库里面创建名为`article`、`category`的两张表

```sql
create table category
(
    id   SERIAL PRIMARY KEY,
    name varchar(100)
);

create table article
(
    title       varchar(100),
    category_id integer references category (id)
)
```

- 查看生成后的模型

```bash
$ python manage.py inspectdb
from django.db import models


class Article(models.Model):
    title = models.CharField(max_length=100, blank=True, null=True)
    category = models.ForeignKey('Category', models.DO_NOTHING, blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'article'


class Category(models.Model):
    name = models.CharField(max_length=100, blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'category'
```

- 导入到文件中

```bash
$ python manage.py inspectdb > models.py
$ cat models.py
from django.db import models


class Article(models.Model):
    title = models.CharField(max_length=100, blank=True, null=True)
    category = models.ForeignKey('Category', models.DO_NOTHING, blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'article'


class Category(models.Model):
    name = models.CharField(max_length=100, blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'category'
```

## 如何将现有数据库视图添加到模型

在SQL中，一切都是表，就算是视图返回的数据其实也是一张表，所以首先创建一个简单的视图

```sql
create view user_list as
select id, username
from auth_user;
```

然后以在我们的应用程序创建的相关模型，并设置`managed = False`和`db_table="user_list"`

```python
class UserList(models.Model):
    username = models.CharField(max_length=100)

    class Meta:
        managed = False
        db_table = "user_list"
```

最后你就可以像使用正常`model`一样使用`UserList`了

```python
>>> from orm.models import UserList
>>> UserList.objects.all()
<QuerySet []>
>>> from django.contrib.auth import get_user_model
>>> User = get_user_model()
>>> User.objects.create(username="user1")
<User: user1>
>>> User.objects.create(username="user2")
<User: user2>
>>> UserList.objects.all()
<QuerySet [<UserList: UserList object (1)>, <UserList: UserList object (2)>]>
>>> UserList.objects.filter(username="user1")
<QuerySet [<UserList: UserList object (1)>]>
>>> UserList.objects.values()
<QuerySet [{'id': 1, 'username': 'user1'}, {'id': 2, 'username': 'user2'}]>
```

## 如何为模型指定表名

在模型中使用`db_table`参数来指定生成数据库表的名称

```python
class Account(models.Model):
    first_name = models.CharField(max_length=100)

    class Meta:
        db_table = "account"
```

这样在数据库中对应的表明就是`account`

## 如何指定模型字段的列名

字段的`db_column`参数可以指定我们在数据库中的字段名

```python
class ColumnName(models.Model):
    a = models.CharField(max_length=40, db_column='column1')
    column2 = models.CharField(max_length=50)
```

## null=True和blank=True之间有什么区别

`null=True`用来定义数据库级别的约束，这表示在数据库中，运行`date`字段设置为`null`

```python
date = models.DateTimeField(null=True)
```

`blank=True`用来约束表单验证，这表示在参数验证的时候`title`可以为空

```python
title = models.CharField(blank=True)
```

同时设置`null=True blank=True`意味着该字段在所有情况下都是可选的

```python
epic = models.ForeignKey(null=True, blank=True)
```

## 如何使用UUID而不是ID作为主键

模型如下

```python
import uuid

from django.db import models


# Create your models here.


class Event(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    details = models.TextField()
```

表结构更新之后我们来测试下

```python
>>> from orm.models import Event
>>> event = Event.objects.create(details="detail")
>>> event
<Event: Event object (1e0fbd5a-88ea-44e4-bfda-ee22d8e26fb4)>
>>> event.pk
UUID('1e0fbd5a-88ea-44e4-bfda-ee22d8e26fb4')
```
