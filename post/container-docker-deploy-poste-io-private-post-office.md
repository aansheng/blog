# 容器化(Docker)部署poste.io私有邮局

在若大的互联网，我们都会通过邮箱注册各种各样的账号，但是很多时候只有一个主邮箱有着诸多不便，所以我们希望，最好是一个平台对应一个email，且无限制。

我在找了很多可以私有部署的邮局之后，最终选择了[poste.io](https://poste.io/)，只看中了一个点，轻量化部署，1G内存完全足矣，而且部署方便，也就意味着迁移方便。

## 环境

下面我将在[HostHatch](https://hosthatch.com/)的VPS上面部署，配置如下：

```bash
1 核
2 GB RAM
```

请注意，一定要确保你的VPS商家开放了25端口，否则搭建不成功，一般发工单都可以解除限制

- 系统

我将采用`CentOS 8`的系统

```bash
$ cat /etc/redhat-release
CentOS Linux release 8.3.2011
$ uname -a
Linux localhost.localdomain 4.18.0-240.15.1.el8_3.x86_64 #1 SMP Mon Mar 1 17:16:16 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

`docker`和`docker-componse`安装请参考[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)和[Install Docker Compose](https://docs.docker.com/compose/install/)。

## [运行poste.io](http://xn--poste-tj4o143b.io/)

- 创建poste.io运行目录

```bash
$ mkdir -p ~/deploy/mail
$ cd ~/deploy/mail
```

- 创建数据存放目录

```bash
$ mkdir data
```

- 创建`docker-compose.yml`配置文件

```bash
$ vim docker-compose.yml
version: "3.8"

services:

  mail:
    image: analogic/poste.io
    container_name: mailserver
    hostname: mail.ansheng.me
    restart: always
    environment:
     - TZ=Asia/Shanghai
     - DISABLE_CLAMAV=TRUE
     - DISABLE_RSPAMD=TRUE
     - VIRTUAL_HOST=mail.ansheng.me
     - HTTPS=OFF
     - HTTP_PORT=10080
     #- HTTPS_PORT=10433
    ports:
     - "25:25"
     - "172.17.0.1:10080:10080"
     #- "172.17.0.1:10433:10433"
     - "110:110"
     - "143:143"
     - "465:465"
     - "587:587"
     - "993:993"
     - "995:995"
    volumes:
     - ./data:/data
```

由于https是在NGINX上面配置的，所以poste.io只运行80端口且绑定在`172.17.0.1:10080`这个docker的host地址。

下面两个配置是关闭反垃圾邮件的功能，实测可以节省很多内存

```bash
- DISABLE_CLAMAV=TRUE
- DISABLE_RSPAMD=TRUE
```

在部署中，HTTPS我们在NGINX中上面配置，为了可以共用80和443，至于那些选项是什么意思，可以参考[Getting started](https://poste.io/doc/)，官方文档会有说明。

- 运行

```bash
$ docker-compose up -d
```

- 查看日志

```bash
$ docker-compose logs -f
# 当出现下面的提示表示启动完成
......
mail_1  | [cont-init.d] done.
mail_1  | [services.d] starting services
mail_1  |
mail_1  |
mail_1  |  Poste.io administration available at <https://172.30.0.2:443> or <http://172.30.0.2:8080>
mail_1  |
mail_1  |
mail_1  | [services.d] done.
```

## 配置DNS解析

官方文档请参考[Configuring DNS](https://poste.io/doc/configuring-dns)，我的域名是放在Cloudflare的，首先我们需要加一条A记录，如下：

![Untitled](/images/2021/04/01/1.png)

增加`imap`、`pop`、`smtp`记录

![Untitled](/images/2021/04/01/2.png)

增加`MX`、`SPF`、`DMARC`

![Untitled](/images/2021/04/01/3.png)

其实我都已经加好了，所以我只是截图把配置发出来，按照上面的配置对照着来就可以

## 配置Nginx

- 安装

```bash
$ dnf install nginx -y
```

- 添加配置

```
$ vim /etc/nginx/conf.d/mail.ansheng.me.conf
server {
    listen 80;
    server_name mail.ansheng.me;

    if ($host = mail.ansheng.me) {
        return 301 https://$host$request_uri;
    }
    return 404;
}

server {
    listen 443 ssl http2;
    server_name   mail.ansheng.me;

    ssl_certificate "/etc/nginx/ssl/mail.pem";
    ssl_certificate_key "/etc/nginx/ssl/mail.key";

    index index.html;

    location / {
        proxy_pass http://172.17.0.1:10080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 43200000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_buffering off;
    }
}
```

我将SSL证书放在了`/etc/nginx/ssl/`这个目录下

- 启动并设置开机启动

```bash
$ systemctl enable --now nginx
```

浏览器访问`https://mail.ansheng.me/`，会让你配置管理员的账号和密码，做对应的设置就可以

![Untitled](/images/2021/04/01/4.png)

## 配置

登陆后台的面板大概长这样

![Untitled](/images/2021/04/01/5.png)

- TLS

选择左侧栏的`System settings` -> `TLS certificate`，[我们需要上传ssl证书到poste.io](http://xn--sslposte-k39ly3a85b4ru23a131d5v6hqshg03c.io/)，否则客户端TLS无法使用

![Untitled](/images/2021/04/01/6.png)

- DKIM

为了防止进入垃圾箱，我们需要添加DKIM的配置，找到`Virtual domains` -> `mail.ansheng.me`请选择自己的域名 -> 点击`create new key`

![Untitled](/images/2021/04/01/7.png)

![Untitled](/images/2021/04/01/8.png)

在到DNS里面添加以下记录

![Untitled](/images/2021/04/01/9.png)

- 收发件测试

配置完成之后我们可以退出登录，然后登陆`https://mail.ansheng.me/webmail/`，发送一封邮件到其他邮箱测试是否可以收到

![Untitled](/images/2021/04/01/10.png)

打开Gmail，很快就会收到邮件，有可能会进垃圾箱，可能解析还没生效

![Untitled](/images/2021/04/01/11.png)

然后我们回复邮件看是否能收到

![Untitled](/images/2021/04/01/12.png)

可以收到回复

![Untitled](/images/2021/04/01/13.png)

- 客户端

如果你需要配置客户端，例如通过Gmail、Foxmail这些第三方的，请参考[Example client settings](https://poste.io/doc/client-settings)，注意host和端口，我在iOS上面的Gmail已经配置成功，可以收发邮件，正常使用。

## 结尾

- 端口说明

|端口|描述|
|:--|:--|
|25	|SMTP - mostly processing incoming mails|
|110|	POP3 - standard protocol for accessing mailbox, STARTTLS is required before client auth|
|143|	IMAP - standard protocol for accessing mailbox, STARTTLS is required before client auth|
|465|	SMTPS - Legacy SMTPs port|
|587|	MSA - SMTP port used primarily for email clients after STARTTLS and auth|
|993|	IMAPS - alternative port for IMAP encrypted since connection|
|995|	POP3S - encrypted POP3 since connections|

其实在了解`poste.io`是怎么运行的原理之后，在研究其他邮局也都是大同小异。