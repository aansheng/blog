# CentOS安装Docker和Docker-componse

Docker属于Linux容器的一种封装，提供简单易用的容器使用接口，而Docker Compose是Docker官方编排项目之一，负责快速的部署分布式应用。

## 环境

```
$ cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)
$ uname -a
Linux sj 4.18.0-193.28.1.el8_2.x86_64 #1 SMP Thu Oct 22 00:20:22 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
$ whoami
root
$ getenforce
Disabled
$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)

```

## Docker

添加docker源

```
$ dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

```

更新缓存

```
$ dnf makecache

```

安装最新版本的docker

```
$ dnf install docker-ce -y

```

启动docker并添加自启动

```
$ systemctl enable --now docker

```

查看docker版本

```
$ docker --version
Docker version 19.03.13, build 4484c46d9d

```

运行`hello-world`镜像来验证`Docker`是否已正确安装

```
$ docker run --rm hello-world

```

通过脚本安装docker

```
$ curl -fsSL <https://get.docker.com> -o get-docker.sh && sh get-docker.sh && rm -f get-docker.sh

```

将admin用户增加到docker组，以让admin用户拥有执行docker的权限

```
$ usermod -aG docker admin

```

卸载docker

```
$ dnf remove docker-ce -y
$ rm -rf /var/lib/docker

```

## Docker-componse

安装curl命令

```
$ dnf install curl -y

```

下载当前稳定版本的Docker Compose

```
$ curl -L "<https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$>(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

添加可执行权限

```
$ chmod +x /usr/local/bin/docker-compose

```

添加软链以让其他普通用户也可以执行

```
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```

查看版本

```
$ docker-compose --version
docker-compose version 1.27.4, build 40524192

```

卸载

```
$ rm -f /usr/bin/docker-compose
$ rm -f /usr/local/bin/docker-compose
```