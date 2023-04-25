# Docker快速入门指南

## 镜像操作

- 搜索镜像

```bash
$ docker search centos
```

- 下载镜像

```bash
$ docker pll centos
```

- 导入镜像

```bash
$ docker load < ./docker/image/centos.tar.gz
```

- 导出镜像

```bash
$ docker save centos > /tmp/centos.tar.gz
```

- 删除镜像

如果这个镜像创建了容器不会被删除

```bash
$ docker rmi centos
```

- 查看当前拥有的镜像

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              a8493f5f50ff        2 weeks ago         192MB
nginx               latest              5766334bdaa0        2 weeks ago         183MB
```

|选项|描述|
|:--|:--|
|REPOSITORY|所属仓库|
|TAG|标签|
|IMAGE ID|进项ID|
|CREATED|创建时间|
|SIZE|镜像创建时间|

- 删除所有的镜像

```bash
$ docker rmi $(docker images -q)
```

## 容器操作

- 启动一个新容器

```bash
$ docker run -i -t centos /bin/bash
```

- 启动一个已经存在的容器

```bash
$ docker start 4253385a9655
```

- 启动一个容器退出时不关闭

```bash
$ docker run --name mydocker -t -i centos /bin/bash
[root@5324897fb034 /]# ps -aux|grep -v "ps -aux" # 可以看到只启动了一个/bin/bash进程
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  11776  3084 ?        Ss   05:49   0:00 /bin/bash
```

|参数|描述|
|:--|:--|
|--name|镜像的名字|
|-t|分配一个伪终端|
|-i|容器标准输入保持打开状态|

通过容器的ID我们可以发现和容器里面的主机名是一致的

```bash
$ docker ps -a | grep "5324897fb034"
5324897fb034        centos              "/bin/bash"              6 minutes ago       Up 6 minutes                                    mydocker
```

- 启动一个容器在退出的时候自动删除这个容器

```bash
$ docker run --rm -it centos /bin/bash
```

- 进入一个已经运行的容器

```bash
# 这种方式启动，如果退出容器，则这个容器就会被退出，如果其他人登陆这个容器，则操作是同步的
$ docker attach 5324897fb034
```

通过`nsenter`访问容器且推出时不销毁容器

```bash
# 获取容器的PID
$ docker inspect --format "{{.State.Pid}}" 5324897fb034
15434
# 指定PID进入容器
$ nsenter -t 15434 -u -i -n -p
```

> nsenter访问的是一个进程的命名空间

- 查看所有的容器

```bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4253385a9655        centos              "/bin/bash"         6 minutes ago       Up 4 minutes                            thirsty_visvesvaraya
```

|选项|描述|
|:--|:--|
|CONTAINER ID|容器的ID|
|IMAGE|通过哪个镜像来运行的|
|COMMAND|最后执行的命令|
|CREATED|创建时间|
|STATUS|状态|
|PORTS|启动的端口|
|NAMES|容器的名称，名称默认是自动生成的|

- 删除一个容器

```bash
$ docker rm 307f7b240273
```

- 删除一个正在运行的容器

```bash
$ docker rm -f 9dc6a6940ce1
```

- 停止所有正在运行的容器

```bash
docker kill $(sudo docker ps -a -q)
```

- 删除所有容器

```bash
$ docker kill $(docker ps -a -q)
```

- 删除状态为退出的所有容器

```bash
$ docker rm $(docker ps -a -f status=exited -qa)
```

## 网络

- 随机端口映射

```bash
# 启动容器
$ docker run -d -P nginx
# 查看Nginx访问日志
$ docker logs c506ed2a4e65
```

- 指定端口映射

```bash
$ sudo docker run -d -p 81:80 nginx
```

## 存储

### 数据卷


|指令|描述|
|:--|:--|
|-v /data|自动生成目标目录|
|-v src :dst :rw |指定目录挂载，并指定挂载目录的权限|

```bash
$ docker run -i -t --name volume-test1 -v /data centos /bin/bash
# ls /data
# 查看挂载的目录
$ docker inspect 80cef0fbd4f1
```

指定目录挂载

```bash
$ sudo docker run -i -t -v /opt:/opt centos   
# df -h | grep "/opt"
/dev/sda2       457G   62G  373G  15% /opt
```

### 数据卷容器

一个容器访问另一个容器的数据卷

```bash
$ docker run --name nfs -it -d -v /data centos
```

|参数|描述|
|:--|:--|
|-d|让容器在后台运行|

启动其他的容器，使用nfs容器里面的卷

```bash
$ docker run -it --name test1 --volumes-from nfs centos
```

## 构建Docker镜像

- 手动构建

```bash
$ docker run -i -t --name webserver centos
# cd /etc/yum.repos.d/
# rpm -ivh https://mirrors.aliyun.com/centos/7.3.1611/os/x86_64/Packages/wget-1.14-13.el7.x86_64.rpm
# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum -y install nginx
# vi /etc/nginx/nginx.conf
daemon off;
# exit
```

```bash
$ docker commit -m "webserver" a6e5ab54bf16 ansheng/webserver:v1
$ docker images | grep "webserver"
webserver           v1                  bd32c3d82378        39 seconds ago      363MB
$ docker run -d -p 80:80 webserver:v1 nginx
$ curl -I 127.0.0.1:82
HTTP/1.1 200 OK
```

- 通过DockerFile构建

```bash
$ cd /tmp/
$ mkdir webserver
$ cd webserver/
$ echo "Hello, World!" > index.html
$ vim Dockerfile
# This docker file
# VERSION
# Author: An Sheng
# Base image
FROM centos

#Maintainer
MAINTAINER ansheng ianshengme@gmail.com

# Commands
RUN rpm -ivh https://mirrors.aliyun.com/centos/7.3.1611/os/x86_64/Packages/wget-1.14-13.el7.x86_64.rpm
RUN wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
RUN yum install nginx -y
ADD index.html /usr/share/nginx/html/index.html
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx"]
$ docker build -t webserver:v2 ./
```

运行容器

```bash
$ docker run -d -p 80:80 webserver:v2
$ curl -I 127.0.0.1:80
HTTP/1.1 200 OK
```

- Dockerfile描述

|指令|描述|
|:--|:--|
|FROM|指定基础镜像|
|MAINTAINER|维护者信息|
|RUN|执行的命令|
|ADD|把本地文件copy到镜像中|
|WORKDIR|指定工作目录|
|VOLUME|目录挂载|
|EXPOSE|启动的端口|
|CMD|最后执行的指令|

## 其他操作

- 时区问题

```bash
[root@d4e104c000c2 ~]# date
Sat Apr 29 08:51:51 UTC 2017
[root@d4e104c000c2 ~]# ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
[root@d4e104c000c2 ~]# date
Sat Apr 29 16:52:28 CST 2017
```

- 字符集问题

```bash
LANG=en_US.UTF-8
```

- 普通用户使用docker

```bash
sudo gpasswd -a ${USER} docker
```
