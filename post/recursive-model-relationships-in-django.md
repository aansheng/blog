# Django中的递归模型关系

在本文中，我们将使用`Django`开发一个人力资源（HR）应用程序，并在员工和管理之间建立递归关系。

## 设置Django项目结构

为了不影响系统环境，你也许会创建一个虚拟环境，当然我已经建立好了虚拟环境，我们开始创建`Django`项目吧

```bash
(test) $ pip install django
```

安装Django后，可以利用`django-admin`生成项目模板，我们称之为`webapp`。

```bash
(test) $ django-admin startproject webapp
```

现在`cd`进入新的`webapp`目录，我们通过`manage.py`脚本提供的`django工具`。我们使用它来创建项目的应用程序，我们将其命名为`hrmgmt`。这将创建另一个名为`hrmgmt`的目录，这是该应用程序的代码所在的位置。

```bash
(test) $ cd webapp/
(test) $ python manage.py startapp hrmgmt
```

最后我们还需要将刚创建的`hrmgmt`APP注册到django项目的APP列表中

```python
# webapp/settings.py
# Application definition

INSTALLED_APPS = [
    ......
    'hrmgmt.apps.HrmgmtConfig'
]
```

## 配置路由

在`webapp/urls.py`中使用以下代码将所有以`/hr`为前缀的路由重定向到`hrmgmt`应用程序中

```python
# webapp/urls.py
from django.conf.urls import url, include  
from django.contrib import admin

urlpatterns = [  
    url(r'^admin/', admin.site.urls),
    url(r'^hr/', include('hrmgmt.urls'))
]
```

在`hrmgmt`应用程序中，创建一个名为`urls.py`的新文件，并放置以下代码。

```python
# hrmgmt/urls.py
from django.conf.urls import url

from . import views

urlpatterns = [
    # /hr/
    url(r'^$', views.index, name='index')
]
```

这将指定一个视图，它将返回所有员工的列表。上面的代码使用正则表达式来表示当从我们的服务器请求`/hr/`的路由时，一个名为`index`的视图函数应该处理该请求并返回一个响应。

## 视图

现在我们来实现前面提到的`index`视图函数来处理对`/hr/`的路由的请求，并返回一个文本响应。

在`hrmgmt/views.py`中包含以下代码：

```python
# hrmgmt/views.py
from django.http import HttpResponse

def index(request):  
    response = "My List of Employees Goes Here"
    return HttpResponse(response)
```

在webapp目录下，启动Django并测试我们已经正确配置了路由和查看功能：

```bash
(test) $ python manage.py runserver
```

现在打开浏览器，输入`http://127.0.0.1:8000/hr/`,就可以看到`My List of Employees Goes Here`。

## 设计模型类

在本节中，我们定义我们的模型类，它将转换成数据库表，所有这些都通过编写Python代码来完成。

在`hrmgmt/models.py`中放置以下代码：

```python
# hrmgmt/models.py
from django.db import models

class Employee(models.Model):  
    STANDARD = 'STD'
    MANAGER = 'MGR'
    SR_MANAGER = 'SRMGR'
    PRESIDENT = 'PRES'

    EMPLOYEE_TYPES = (
        (STANDARD, 'base employee'),
        (MANAGER, 'manager'),
        (SR_MANAGER, 'senior manager'),
        (PRESIDENT, 'president')
    )

    role = models.CharField(max_length=25, choices=EMPLOYEE_TYPES)
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    manager = models.ForeignKey('self', null=True, related_name='employee')

    def __str__(self):
        return "<Employee: {} {}>".format(self.first_name, self.last_name)

    def __repr__(self):
        return self.__str__()
```

在上面的代码中，我们创建了一个django的model`Employee`，该类具有Django的ORM访问数据库的功能。

接下来是四个类字段的定义，它们是常量（STANDARD，MANAGER，SR_MANAGER，PRESIDENT）以及它们用于进一步定义元组类字段常量的定义。这些类似于枚举，它们指定了员工可以承担的不同角色。

接下来`first_name`，`last_name`字段被定义为最大长度为100个字符的字符。

最后一个字段定义员工与管理之间递归关系的外键，这意味着Django对继承的模型所做的隐式自动递增整数`id`列`django.db.models.Model`将作为同一个类（或表）的外键值提供。

## 生成数据表

运行以下命令创建所有Django应用程序使用的表。默认情况下，数据库使用的是`sqlite`。

```bash
(test) $ python manage.py makemigrations
(test) $ python manage.py migrate
```

您可以通过运行以下命令来查看创建表的实际SQL：

```bash
(test) $ python manage.py sqlmigrate hrmgmt 0001
BEGIN;
--
-- Create model Employee
--
CREATE TABLE "hrmgmt_employee" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "role" varchar(25) NOT NULL, "first_name" varchar(100) NOT NULL, "last_name" varchar(100) NOT NULL, "manager_id" integer NULL REFERENCES "hrmgmt_employee" ("id"));
CREATE INDEX "hrmgmt_employee_manager_id_43028de6" ON "hrmgmt_employee" ("manager_id");
COMMIT;
```

## 用Django Shell创建数据

输入以下命令，进入带`Django`环境的Python解释器：

```bash
(test) $ python manage.py shell
```

下面的代码创建了四名员工。

```python
>>> from hrmgmt.models import Employee
>>> janeD = Employee.objects.create(first_name='Jane', last_name='Doe', role=Employee.PRESIDENT)
>>> johnD = Employee.objects.create(first_name='John', last_name='Doe', role=Employee.MANAGER, manager=janeD)
>>> joeS = Employee.objects.create(first_name='Joe', last_name='Scho', role=Employee.STANDARD, manager=johnD)
>>> johnB = Employee.objects.create(first_name='John', last_name='Brown', role=Employee.STANDARD, manager=johnD)
```

## 创建显示模板

在`hrmgmt`目录中创建另一个名为`templates`的目录。然后在`templates`目录下再创建一个名为`hrmgmt`的目录。最后在`hrmgmt/templates/hrmgmt`目录下创建一个名为`index.html`的HTML文件。在这个文件中，我们将编写代码来构建我们的员工列表。

代码如下：

```html
<!-- hrmgmt/templates/hrmgmt/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Employee Listing</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/4.0.0-beta/css/bootstrap.min.css" integrity="sha384-/Y6pD6FV/Vv2HJnA6t+vslU6fwYXjCFtcEpHbNJ0lyAFsXTsjBbfaDjzALeQsN6M" crossorigin="anonymous">
    <script src="https://cdn.bootcss.com/bootstrap/4.0.0-beta/js/bootstrap.min.js" integrity="sha384-h0AbiXch4ZDo7tp9hKZ4TsHbi047NrKGLO3SEJAg45jXxnGIfYzk4Si90RDIqNm1" crossorigin="anonymous"></script>
</head>
<body>
<div class="container">
    <div class="row">
        <div class="col-md-12">
            <h1>Employee Listing</h1>
        </div>
    </div>
    <div class="row">
        <dov class="col-md-12">
            <table class="table table-striped">
                <thead class="thead-inverse">
                <tr>
                    <th>Employee ID</th>
                    <th>First Name</th>
                    <th>Last Name</th>
                    <th>Role</th>
                    <th>Manager</th>
                </tr>
                </thead>
                <tbody class='table-striped'>
                {% for employee in employees %}
                    <tr>
                        <td>{{ employee.id }}</td>
                        <td>{{ employee.first_name }}</td>
                        <td>{{ employee.last_name }}</td>
                        <td>{{ employee.get_role_display }}</td>
                        <td>{% if employee.manager %}{{ employee.manager.first_name }} {{ employee.manager.last_name }}{% endif %}</td>
                    </tr>
                {% endfor %}
                </tbody>
            </table>
        </dov>
    </div>
</div>
<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
<script src="https://cdn.bootcss.com/popper.js/1.11.0/umd/popper.min.js" integrity="sha384-b/U6ypiBEHpOf/4+1nzFpr53nxSS+GLCkfwBdFNTxtclqqenISfwAzpKaMNFNmj4" crossorigin="anonymous"></script>
</body>
</html>
```

我们需要在`view`中获取所有的员工数据，然后传递给html模板中。

```python
# hrmgmt/views.py
from django.shortcuts import render

from .models import Employee


def index(request):
    employees = Employee.objects.order_by('id').all()
    context = {'employees': employees}
    return render(request, 'hrmgmt/index.html', context)
```

再次启动`Django`服务，打开浏览器输入`http://127.0.0.1:8000/hr/`,你将会看到如下截图：

![1507689271591157](/images/2017/10/1507689271591157.png)
