# Gogs Docker化部署以及配置LDAP(FreeIPA)认证

[Gogs](https://gogs.io/)是一款极易搭建的自助Git服务，相比GitLab，它已经非常轻量级了。

## 安装

关于FreeIPA Server端的部署你可以参考[CentOS部署FreeIPA Server和FreeIPA Client](https://ansheng.me/centos-deploys-freeipa-server-and-freeipa-client/)，这次使用的FreeIPA Server端IP为`95.179.246.183`.

创建部署目录

```
$ mkdir -p ~/deploy/git

```

创建`docker-compose.yml`文件

```
$ vi ~/deploy/git/docker-compose.yml
version: "3.8"

services:
  gogs:
    image: gogs/gogs
    restart: always
    # 应该添加DNS记录的，但是我在服务器上面没有调通
    # dns:
    #   - 95.179.246.183
    #   - 8.8.8.8
    # 添加hosts记录
    extra_hosts:
      - "ipa.corp.ansheng.me:95.179.246.183"
    ports:
      # SSH 端口
      - "10022:22"
      # Web端口
      - "10080:3000"
    volumes:
      # 数据挂在目录
      - ./data:/data

```

通过docker-compose运行服务

```
$ cd ~/deploy/git/
$ docker-compose up -d

```

当启动完成之后我们打开：`http://45.77.54.234:10080/`进行初始化操作，因为我的Gogs服务器IP是45.77.54.234，请把这个IP替换为你自己的服务器IP

![Untitled](/images/2020/11/20/1.png)

为了方便测试，上面的数据库我用的是`SQLite3`，域名填写当前的IP地址，SSH端口号填写我们映射出来的10022端口，应用URL改为当前页面的URL，我这里还创建了一个管理员账号并且禁止注册。

为了方便调试，我们把gogs的日志级别改为`TRACE`模式

```
vi ~/deploy/git/data/gogs/conf/app.ini
......
LEVEL     = TRACE
......
$ docker-compose restart

```

## 配置LDAP(FreeIPA)认证

我们需要切换到FreeIPA Server端创建一个gogs用户和staff组

- 创建staff组

只要加入到改组的用户都可以登陆gogs

```
$ ipa group-add --desc="Staff" staff
-----------
已添加组"staff"
-----------
  组名: staff
  描述: Staff
  GID: 139600003

```

- 导入LDIF

创建gogs.ldif文件，这里的配置只需要修改DC部分

```
$ vi gogs.ldif
dn: uid=gogs,cn=sysaccounts,cn=etc,dc=corp,dc=ansheng,dc=me
changetype: add
objectclass: account
objectclass: simplesecurityobject
uid: gogs
userPassword: secure-password
passwordExpirationTime: 20380119031407Z
nsIdleTimeout: 0

```

通过ldapmodify导入，系统将提示输入目录管理员密码：

```
$ ldapmodify -h ipa.corp.ansheng.me -p 389 -x -D "cn=Directory Manager" -W -f gogs.ldif
Enter LDAP Password: # 输入目录管理员密码
adding new entry "uid=gogs,cn=sysaccounts,cn=etc,dc=corp,dc=ansheng,dc=me"

```

- 创建gogs用户，用来做ldap查询用

```
$ ipa user-add gogs --first=Gogs --last=Gogs --random
-----------
已添加用户"gogs"
-----------
  用户登录名: gogs
  名: Gogs
  姓: Gogs
  全名: Gogs Gogs
  显示名称: Gogs Gogs
  名字的首字母: GG
  主目录: /home/gogs
  GECOS: Gogs Gogs
  登录shell: /bin/sh
  主机名: gogs@CORP.ANSHENG.ME
  主体别名: gogs@CORP.ANSHENG.ME
  User password expiration: 20201120090543Z
  邮件地址: gogs@corp.ansheng.me
  随机密码: 5Bh~{GfM@::nA5stMfaPl9
  UID: 139600004
  GID: 139600004
  密码: True
  组成员: ipausers
  Kerberos密码可用: True
# 尝试登陆然后更新密码
$ ssh gogs@ipa.corp.ansheng.me
Password: # 输入刚才生成的随机密码
Password expired. Change your password now.
Current Password:
New password: # 新密码为secure-password
Retype new password:
Creating home directory for gogs.

```

- 添加测试用户

创建gogs_test_user用户

```
$ ipa user-add gogs_test_user --first=gogs --last=gogs --cn=GG --random
---------------------
已添加用户"gogs_test_user"
---------------------
  用户登录名: gogs_test_user
  名: gogs
  姓: gogs
  全名: GG
  显示名称: gogs gogs
  名字的首字母: gg
  主目录: /home/gogs_test_user
  GECOS: gogs gogs
  登录shell: /bin/sh
  主机名: gogs_test_user@CORP.ANSHENG.ME
  主体别名: gogs_test_user@CORP.ANSHENG.ME
  User password expiration: 20201120090127Z
  邮件地址: gogs_test_user@corp.ansheng.me
  随机密码: 2Rj?)7v2ICrH8G}P:-^!b?
  UID: 139600001
  GID: 139600001
  密码: True
  组成员: ipausers
  Kerberos密码可用: True

```

通过随机密码登录，然后重置密码

```
$ ssh gogs_test_user@ipa.corp.ansheng.me
Password:
Password expired. Change your password now.
Current Password:
New password:
Retype new password:
Creating home directory for gogs_test_user.

```

将gogs_test_user和admin用户加入staff组

```
$ ipa group-add-member staff --users={gogs_test_user,admin}
  组名: staff
  描述: Staff
  GID: 139600003
  成员用户: gogs_test_user, admin
---------
已添加的成员数 2
---------

```

- 创建gogs ldap配置文件

下面的操作将在gogs服务器上执行

```
$ mkdir ~/deploy/git/data/gogs/conf/auth.d/
$ vi ~/deploy/git/data/gogs/conf/auth.d/ldap_bind_dn.conf
# This is an example of LDAP (BindDN) authentication
#
id           = 101
type         = ldap_bind_dn
name         = Staff
is_activated = true
is_default   = true

[config]
host               = ipa.corp.ansheng.me
port               = 636
# 0 - Unencrypted, 1 - LDAPS, 2 - StartTLS
security_protocol  = 1
skip_verify        = true
bind_dn            = uid=gogs,cn=users,cn=accounts,dc=corp,dc=ansheng,dc=me
bind_password      = secure-password
user_base          = cn=users,cn=accounts,dc=corp,dc=ansheng,dc=me
attribute_username = uid
attribute_name     = givenName
attribute_surname  = sn
attribute_mail     = mail
attributes_in_bind = true
filter             = (&(memberOf=cn=staff,cn=groups,cn=accounts,dc=corp,dc=ansheng,dc=me)(uid=%s))
admin_filter       = (memberOf=cn=admins,cn=groups,cn=accounts,dc=corp,dc=ansheng,dc=me)
group_enabled      = false
group_dn           =
group_filter       =
group_member_uid   =
user_uid           =

```

上面的参数请确保每个都填写正确，filter规则表示这个用户在staff组则允许登陆，admin规则表示这个用户要在admins组才会是gogs是管理员。

- 重启服务

```
$ docker-compose restart

```

然后我们用`root`用户登陆Gogs，右上角管理面板->认证源管理就可以看到我们刚添加的配置文件了

![Untitled](/images/2020/11/20/2.png)

为了测试配置是否成功，我们退出root用户，然后重新登陆，在登陆界面中我们选择`LDAP BindDN`方式

![Untitled](/images/2020/11/20/3.png)

如果登陆成功，恭喜，配置成功，这里如果用`gogs_test_user`用户登陆则gogs会创建一个普通用户相对应，如果用admin用户登陆则会报500错误，admin被gogs占用了，所以不要用这个用户名，如果你要测试你需要创建一个用户并把这个用户添加到staff组和admin组才可以。

这里还有个坑要注意下，Gogs不会在每次登陆的时候同步用户信息，比如：当把sa用户添加到admins组，sa用户第一次登陆的时候是gogs的管理员，过段时间之后你在FreeIPA里面把sa用户从admins组里面剔掉了，你再用sa用户登陆gogs，依旧是gogs的管理员，你可以参考这个issues[LDAP user information doesn't update](https://github.com/gogs/gogs/issues/4701)。

一个小小的功能，在FreeIPA Server上面获取用户gogs用户的DN，你能把这个看懂了，ldap_bind_dn.conf配置文件中的filter对你来说就不在话下了

```
$ ldapsearch -x -H ldaps://ipa.corp.ansheng.me -w "secure-password" -D "uid=gogs,cn=users,cn=accounts,dc=corp,dc=ansheng,dc=me" -b "uid=gogs,cn=users,cn=accounts,dc=corp,dc=ansheng,dc=me"

```

## 参考文献

- [ldap_bind_dn.conf.example](https://github.com/gogs/gogs/blob/f2ecfdc96a338815ffb2be898b3114031f0da48c/conf/auth.d/ldap_bind_dn.conf.example)
- [ldap_simple_auth.conf.example](https://github.com/gogs/gogs/blob/f2ecfdc96a338815ffb2be898b3114031f0da48c/conf/auth.d/ldap_simple_auth.conf.example)
- [Not able to connect with FreeIPA LDAP](https://github.com/gogs/gogs/issues/5743)