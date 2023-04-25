# Python项目整合PyOTP+Google Authenticator

[PyOTP](https://github.com/pyauth/pyotp)主要是用來生成与验证一次性密码(OTP, One-Time Password)的Python库，下面是官方介绍


> PyOTP is a Python library for generating and verifying one-time passwords. It can be used to implement two-factor (2FA) or multi-factor (MFA) authentication methods in web applications and in other systems that require users to log in.

首先需要安装`PyOTP`

```bash
pip install pyotp
# 这里安装的版本是2.3.0
```

PyOTP提供2种One-Time Password可以使用:

- HOTP(HMAC-Based One-Time Password)
- TOTP(Time-Based One-Time Password)

## HOTP

基于HMAC的一次性密码，所谓一次性，自然是生成一个验证码这个验证码只能用一次啦，下面来看看示例

```python
import pyotp

# pyotp提供了一个辅助函数随机生成一个16个字符的base32密钥
# SECRET_KEY = pyotp.random_base32()  # => JTWXYIZEIMMFF4IM
SECRET_KEY = 'base32secret3232'  # 方便测试我这里使用固定的密钥

# 创建hotp对象
hotp = pyotp.HOTP(SECRET_KEY)

# 生成三个验证码
hotp.at(0)  # =>验证码：260182
hotp.at(1)  # =>验证码：055283
hotp.at(1401)  # =>验证码：316439

# 服务上面验证
hotp = pyotp.HOTP(SECRET_KEY)  # SECRET_KEY是共享密钥
hotp.verify('316439', 1401)  # 验证通过True
hotp.verify('316439', 1402)  # 验证失败False
```

## TOTP

基于时间的一次性密码，比如我们本文中使用到的`Google Authenticator`，每次生成的验证码有效期就只有30秒，30秒之后就过期无法使用，下面来看看示例

```python
import time

import pyotp

SECRET_KEY = 'base32secret3232'

# 生成TOTP对象
totp = pyotp.TOTP('base32secret3232')
# 获取验证码
totp.now()  # => '492039'

# OTP verified for current time
totp.verify('492039')  # => True
time.sleep(30)
totp.verify('492039')  # => False，因超过30秒被视为验证未通过
```

## 整合Google Authenticator

`PyOTP`提供生成`Google Authenticator URI`功能，让我们更轻松的整和`Google Authenticator`

```python
import pyotp

SECRET_KEY = 'base32secret3232'

# 创建TOTP对象
totp = pyotp.totp.TOTP(SECRET_KEY)

# 生成QrCode链接，你需要将以下内容生成一个二维码，然后使用Google Authenticator App进行绑定
qr_code_text = totp.provisioning_uri(name='alice@google.com', issuer_name='Secure App')
# qr_code_text = otpauth://totp/Secure%20App:alice%40google.com?secret=base32secret3232&issuer=Secure%20App

# 验证
totp.verify('047374')  # => True
```

推荐两个插件，一个是Chorme版本的[Google Authenticator](https://authenticator.cc/)，另外一个是根据[文本生成二维码](https://cli.im/text).
