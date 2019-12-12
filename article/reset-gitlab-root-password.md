# 重置GitLab root密码

启动Ruby on Rails控制台

```bash
gitlab-rails console -e production
```

等待控制台加载完毕。

通过`ID`查找用户

```bash
user = User.where(id: 1).first
```

或者通过邮箱查查找到用户

```bash
user = User.find_by(email: 'admin@ansheng.me')
```

然后更改密码

```bash
user.password = 'secret_pass'
user.password_confirmation = 'secret_pass'
```

保存更改

```bash
user.save!
```

退出控制台，然后尝试使用新密码登录
