# 在Django应用程序中发送电子邮件

在Django应用程序中发送电子邮件最常见的用例是密码重置、帐户激活和发送与您的应用程序相关的一般通知。

## 配置Django发送电子邮件

要配置您的Django应用程序，添加下面的参数到你`settings.py`：

```python
# 主机
EMAIL_HOST = "smtp.sina.com"
# 端口
EMAIL_PORT = 25
# 发件人邮箱
EMAIL_HOST_USER = "anshengme@sina.com"
# 密码
EMAIL_HOST_PASSWORD = "ansheng.me"
# 是否使用https
EMAIL_USE_TLS = False
# 发件人
EMAIL_FROM = "安生"
```

## 发送邮件

先看看`send_mail()`方法提供了那些参数：

|参数|描述|
|:--|:--|
|subject|邮件标题|
|message|邮件正文|
|from_email|发送者|
|recipient_list|收件人列表|
|fail_silently|布尔值，|
|auth_user|用于向SMTP服务器进行身份验证的可选用户名，如果未提供此项，Django将使用`EMAIL_HOST_USER`设置的值|
|auth_password|用于向SMTP服务器进行身份验证的可选密码，如果未提供此项，Django将使用`EMAIL_HOST_PASSWORD`设置的值|
|connection|用于发送邮件的可选电子邮件后端，如果未指定，将使用默认后端的实例|
|html_message|如果提供了`html_message`，则生成的电子邮件将是一个多部分/替代电子邮件，其消息为`text/plain`内容类型，`html_message`为`text/html`内容类型。|

然后我们进入带`django shell`环境变量的python解释器，然后发送一个邮件试试？

```bash
$ python3 manage.py shell

In [1]: from django.core.mail import send_mail

In [2]: send_mail("这是邮件标题", "这是邮件主体", 'anshengme@sina.com', ['ianshengme@gmail.com'])

# 返回值将是成功传递的消息的数量（可以是0或1，因为它只能发送一个消息）
Out[2]: 1
```

打开接收邮件的邮箱看看是否已经接收到邮件了？

![1484632642](/images/2017/01/1484632642.png)

## 同时发送多封电子邮件

`send_mass_mail()`所提供的参数值

|属性|描述|
|:--|:--|
|datatuple|接收一个一个元组，每个元素都是`(subject, message, from_email, recipient_list)`这种格式|

小栗子

```python
In [1]: message1 = ("这是第一封邮件标题", "这是第一封邮件主体", 'anshengme@sina.com', ['ianshengme@gmail.com'])

In [2]: message2 = ("这是第二封邮件标题", "这是第二封邮件主体", 'anshengme@sina.com', ['ianshengme@gmail.com'])

In [3]: from django.core.mail import send_mass_mail

In [4]: send_mass_mail((message1,message2),fail_silently=False)
Out[4]: 2
```

返回值将是已成功发送邮件的消息数。

![1484632655](/images/2017/01/1484632655.png)