# Django REST framework Token认证不过期的解决方法

首先你需要知道在REST框架中的Token认证不像Session认证一样，它是没有办法设置过期时间的，但是有时我们需要对Token做过期验证，比如说用户在A设备登陆之后获取一个Token，如果用户在没有清空浏览器缓存的情况下，Token将一直保存在缓存中，一周后在访问依旧有效，但我们并不希望这样，我们觉得Token认证应该和Session一样都有过期时间，过期之后作废。

那么如何解决这个问题呢？首先呢，需要看一下`TokenAuthentication`这个模块的源码，里面有一段源码是这样的：

```python
def authenticate_credentials(self, key):  # key=前端在请求头传过来的Token
    model = self.get_model()  # 获取Token模块
    try:
        token = model.objects.select_related('user').get(key=key)  # 通过key获取Token
    except model.DoesNotExist:
        raise exceptions.AuthenticationFailed(_('Invalid token.'))  # 如果没有就报错

    if not token.user.is_active: # 如果用户没有激活也报错
        raise exceptions.AuthenticationFailed(_('User inactive or deleted.'))

    return (token.user, token) # 然后把Token和登录的用户返回给View
```

上面的代码比较简单，仔细观看一下`TokenAuthentication`模块就知道是怎么回事，但我们要解决的是每次验证成功之后都要进行一次过期验证，那么应该如何做呢？其实很简单，只需要扩展`TokenAuthentication`模块就可以了，兴建一个`token.py`文件，代码如下：

```python
from rest_framework.authentication import TokenAuthentication
from django.utils.translation import ugettext_lazy as _
from rest_framework import exceptions
from django.utils import timezone
from datetime import timedelta
from django.conf import settings


class ExpiringTokenAuthentication(TokenAuthentication):
    def authenticate_credentials(self, key):
        model = self.get_model()
        try:
            token = model.objects.select_related('user').get(key=key)
        except model.DoesNotExist:
            raise exceptions.AuthenticationFailed(_('Invalid token.'))
        if not token.user.is_active:
            raise exceptions.AuthenticationFailed(_('User inactive or deleted.'))

        if timezone.now() > (token.created + timedelta(days=settings.TOKEN_LIFETIME)):  # 重点就在这句了，这里做了一个Token过期的验证，如果当前的时间大于Token创建时间+7天，那么久返回Token已经过期
            raise exceptions.AuthenticationFailed(_('Token has expired'))

        return (token.user, token)
```

> TOKEN_LIFETIME = 7

但是并没有结束，我们还需要修改`login View`，来确保如果过期都会刷新Token和更新时间，你可能需要添加如下代码：

```python
 if user is not None:
     token, has_created = Token.objects.get_or_create(user=user)
     if timezone.now() > (token.created + timedelta(days=TOKEN_LIFETIME)):
         Token.objects.filter(user=user).update(key=token.generate_key(), created=timezone.now())
```

这是一个真实存在的问题，如果可以，我推荐你看下[这篇文章](http://stackoverflow.com/questions/14567586/token-authentication-for-restful-api-should-the-token-be-periodically-changed/15380732#15380732)。

最后，你在View中使用Token验证的时候只需要把`TokenAuthentication`替换成`ExpiringTokenAuthentication`即可。