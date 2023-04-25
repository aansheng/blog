# Docker安装及配置GitLab

`GitLab`和`GitHub`一样，都是基于`Git`开发了一些功能，然后商业化的产品，它俩唯一的不同之处在于`GitLab`可以私有化部署，这也就导致受到了很多企业的青睐。

本篇文章主要介绍如何以`Docker`化的方式安装GitLab，以及简单的配置，使用的是`GitLab-CE社区版`，操作系统`CentOS 7`。

## 环境

- 系统更新并重启

```bash
$ sudo yum update -y
$ sudo reboot
```

- 环境查看

```bash
$ hostname
gitlab
$ uname -a
Linux gitlab 3.10.0-957.27.2.el7.x86_64 #1 SMP Mon Jul 29 17:46:05 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
$ whoami
ansheng
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ getenforce
Disabled
$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```

## 安装

- 安装docker
 
```bash
$ curl -sSL https://get.docker.com/ | sudo sh
$ sudo systemctl enable --now docker
```

- 运行GitLab

```bash
$ sudo docker run --detach \
  --hostname git.ansheng.me \
  --publish 443:443 \
  --publish 80:80 \
  --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

上面的配置中，请根据自己的实际情况进行调整，GitLab产生的数据都将存储在`/srv/gitlab/`目录下，如果不需要SSL是不用开放443端口的，因为系统本身占用了22端口，所以我把默认的22改成了`8022`。

- 数据目录介绍

|本地位置|容器位置|作用|
|:--|:--|:--|
|/srv/gitlab/data|/var/opt/gitlab|数据|
|/srv/gitlab/logs|/var/log/gitlab|日志|
|/srv/gitlab/config|/etc/gitlab|配置文件|

- 访问测试

启动之后可能需要很长时间才能运行起来，你可以通过`sudo docker logs -f gitlab`来跟踪日志，第一次访问的时候系统会要求设置管理员密码

![gitlab-set-password](/images/2019/08/gitlab-set-password.png)

设置完密码之后进行登陆，默认的用户名是root

![gitlab-login](/images/2019/08/gitlab-login.png)

登陆成功之后，如下是默认的首页，至此，安装部分以及完成。

![gitlab-home](/images/2019/08/gitlab-home.png)

## 创建一个项目试试

接着上图，我们点击首页上面的`Create a project`来创建一个项目

![gitalab-create-project](/images/2019/08/gitalab-create-project.png)

如上，我们创建了一个私有的项目，项目命名为`caca`，隶属人`root`下，我这里只是作为演示，生产环境中是不会使用root用户的，毕竟不太安全，创建完成之后会进入项目首页，如下：

![gitlab-project-home](/images/2019/08/gitlab-project-home.png)

然后我们点击右上角的Clone，尝试把项目下载下来

![gitlab-project-clone](/images/2019/08/gitlab-project-clone.png)

复制`Clone with SSH`下的指令`git clone ssh://git@git.ansheng.me/root/caca.git`，这条指令默认使用的是22端口，但是我们在运行gitlab时指定的端口是8022，所以我们还需要进行一些配置才能够正确的clone项目。

## 配置

容器使用的是官方`Omnibus GitLab`软件包，配置文件默认在`/etc/gitlab/gitlab.rb`中，若要修改配置，我们首先需要进入容器

```bash
$ sudo docker exec -it gitlab /bin/bash
root@git:/#
```

- 基本的配置

```bash
$ vim /etc/gitlab/gitlab.rb
# 默认使用的是22端口，我们需要更改为8022
gitlab_rails['gitlab_shell_ssh_port'] = 8022
# 如果你更换了域名，需要将下面的配置改为新域名，这里不需要更改
# external_url 'http://newgit.ansheng.me'
```

> 当然你也可以编辑`/srv/gitlab/config/gitlab.rb`这个配置文件

然后退出并重启容器使配置生效

```bash
$ sudo docker restart gitlab
```

再次刷新页面，复制`Clone with SSH`链接，发现端口以及改成了8022

![gitlab-project-change-ssh](/images/2019/08/gitlab-project-change-ssh.png)

然后你就可以愉快的把项目`clone`到本地玩耍了！

- 配置邮件

邮件还是挺重要的，修改密码，消息通知等都会用到邮件，若要配置邮件，进入gitlab容器，然后打开配置文件

```bash
$ vim /etc/gitlab/gitlab.rb
# 增加如下配置
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "xxx@gmail.com"
# 这里的密码是应用专用密码，不是你登陆gmail的密码
gitlab_rails['smtp_password'] = "xxxx"
gitlab_rails['smtp_domain'] = "smtp.gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'peer' 
```

配置修改完之后需要reconfig下，执行如下指令

```bash
$ gitlab-ctl reconfigure
```

然后进入gitlab的console来发送一封测试邮件

```bash
$ gitlab-rails console
Notify.test_email('ooo@gmail.com', 'Message Subject', 'Message Body').deliver_now
```

打开你的邮箱，你就可以收到如下的测试邮件

![gitlab-send-mail](/images/2019/08/gitlab-send-mail.png)

- 关闭注册

在登陆的时候不知道你有没有发现，旁边还有一个注册栏

![gitlab-register](/images/2019/08/gitlab-register.png)

作为一个私有的git，有时我们是不希望别人进行注册的，所以我们需要关闭注册功能，若要添加用户，只能让管理员手动添加。

关闭注册功能需要使用root用户，以root用户登陆之后执行如下操作

1. 点击顶部栏的`Admin Area`；
2. 点击左侧栏的`Settings`；
3. 找到`Sign-up Restrictions`设置；
4. 取消勾选`Sign-up enabled`并点击下方的保存；

最后再次打开你的登陆页面就会发现已经没有注册功能了

![gitlab-login](/images/2019/08/gitlab-login.png)

## 更新

GitLab更新还是挺勤快的....，还是使用新版本吧

- 停止正在运行的容器

```bash
$ sudo docker stop gitlab
```

- 删除现有容器

```bash
$ sudo docker rm gitlab
```

- 拉取最新的image

```bash
$ sudo docker pull gitlab/gitlab-ce:latest
```

- 使用先前指定的选项再次创建容器

```bash
$ sudo docker run --detach \
  --hostname git.ansheng.me \
  --publish 443:443 \
  --publish 80:80 \
  --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

然后经过漫长的等待，再次打开会发现版本已经是最新的了，然后打开顶部栏的`Admin Area`，有一个栏目Components，会显示你当前gitlab的组件版本

![gitlab-admin-components](/images/2019/08/gitlab-admin-components.png)

当然我们刚刚升级完，肯定是最新的咯

## 参考文献

- [GitLab Docker images](https://docs.gitlab.com/omnibus/docker/)
- [SMTP settings](https://docs.gitlab.com/omnibus/settings/smtp.html)