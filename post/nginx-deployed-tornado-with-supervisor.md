# Nginx结合Supervisor部署Tornado

### Supervisor

`Supervisor`是使用Python开发的一款进程管理工具，不能在`Windows`上运行，至少现在是不可以，它主要是在类UNIX上管理进程，当然，它主要管理的是前台进程而不是后台进程，

主要应用场景如一个进程需要对外提供服务并且要一直运行着，但是由于一些未知的因素进程挂掉了，这个时候如果这个进程被`Supervisor`所管理，那么当进程挂掉的时候，`Supervisor`就会自动启动它，继续对外提供服务，否则进程挂掉了就需要我们手动启动了。

操作系统环境如下

```bash
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.3.1611 (Core) 
[root@localhost ~]# uname -a
Linux localhost.localdomain 3.10.0-514.2.2.el7.x86_64 #1 SMP Tue Dec 6 23:06:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /proc/cpuinfo | grep "processor"
processor	: 0
```

安装Supervisor

```bash
[root@localhost ~]# yum -y install supervisor
```

生成配置文件

```bash
[root@localhost ~]# tail -2 /etc/supervisord.conf 
[include]
# 这里是说只要这个目录下面以.ini结尾的文件都会被supervisor加载
files = supervisord.d/*.ini
```

创建`tornado`进程配置文件


```bash
[root@localhost ~]# vim /etc/supervisord.d/tornados.ini
# 为了方便管理，增加一个tornado组
[group:tornados]
programs=tornado-0,tornado-1

# 分别定义三个tornado的进程配置
[program:tornado-0]
# 进程要执行的命令
command=python /root/tornado/hello.py --port=8000
directory=/root/tornado/
user=root
# 自动重启
autorestart=true
redirect_stderr=true
# 日志路径
stdout_logfile=/root/tornado/tornado0.log
loglevel=info


[program:tornado-1]
command=python /root/tornado/hello.py --port=8001
directory=/root/tornado/
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/root/tornado/tornado1.log
loglevel=info
```

创建`tornado`启动程序

```bash
[root@localhost ~]# mkdir tornado
[root@localhost ~]# cd tornado/
[root@localhost tornado]# vim hello.py
#!/usr/bin/env python
# _*_coding:utf-8 _*_

import tornado.ioloop
import tornado.web
import sys

PORT = sys.argv[1]

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write(PORT)

application = tornado.web.Application([
    (r'/', MainHandler),
])

if __name__ == "__main__":
    application.listen(PORT)
    tornado.ioloop.IOLoop.instance().start()
```
启动试试

```bash
[root@localhost tornado]# python3.4 hello.py 8000
```
打开另一个`shell`窗口`curl`下看看有没有返回值

```bash
[root@localhost ~]# curl 127.0.0.1:8000
# 有返回启动时指定的端口代表tornado没问题
8000
```

创建`www`用户

```bash
[root@localhost tornado]# useradd -s /sbin/nologin -M www
```

启动Supervisor

```bash
# 启动的时候需要指定配置文件
[root@localhost tornado]# supervisord -c /etc/supervisord.conf
```

验证启动是否成功

```bash
# 看进程
[root@localhost tornado]# ps -ef | grep "supervisord" | grep -v "grep"
root       1151      1  0 15:50 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c /etc/supervisord.conf
```

访问`tornado`

```bash
[root@localhost tornado]# curl 127.0.0.1:8000
8000
[root@localhost tornado]# curl 127.0.0.1:8001
8001
```

### 配置Nginx

先安装`Nginx`，我这里的配置就简单点，一路往下

```bash
[root@localhost tornado]# cd ../
# 安装编译Nginx所依赖的包
[root@localhost ~]# yum -y install pcre pcre-devel zlib zlib-devel 
```
编译安装Nginx
```bash
[root@localhost ~]# wget http://nginx.org/download/nginx-1.10.2.tar.gz
[root@localhost ~]# tar xf nginx-1.10.2.tar.gz 
[root@localhost ~]# cd nginx-1.10.2
[root@localhost nginx-1.10.2]# ./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--user=nginx \
--group=nginx
[root@localhost nginx-1.10.2]# make
[root@localhost nginx-1.10.2]# make install
```
编辑Nginx配置文件
```
[root@localhost nginx-1.10.2]# cd /etc/nginx/
[root@localhost nginx]# vim nginx.conf
user  www;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    upstream tornados{
        server 127.0.0.1:8000;
        server 127.0.0.1:8001;
    }
    server {
        listen       80;
        server_name  localhost;

    location / {
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        # 把请求方向代理传给tornado服务器，负载均衡
        proxy_pass http://tornados;
      }
    }
}
```
检测与法是否有错误
```
[root@localhost nginx]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
启动Nginx
```
[root@localhost nginx]# nginx 
# 查看启动状态
[root@localhost nginx]# netstat -tlnp | grep "80"
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      6011/nginx: master
```
访问80端口，这个时候发现已经打通了，nginx的调度算法也是轮训的。
```
[root@localhost nginx]# curl 127.0.0.1
8000
[root@localhost nginx]# curl 127.0.0.1
8001
[root@localhost nginx]# curl 127.0.0.1
8000
[root@localhost nginx]# curl 127.0.0.1
8001
...
```
先写这么多吧，冻死了，配置文件什么的都比较少，这个要根据具体业务具体优化的，特别是nginx，这个要问问运维的同学了。。