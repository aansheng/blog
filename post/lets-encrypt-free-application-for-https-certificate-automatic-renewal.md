# Let's Encrypt免费申请https证书+自动续费

[Let's Encrypt](https://letsencrypt.org/)是一个免费，自动化和开放的证书颁发机构，主要就是为了推进让大家都使用https，毕竟之前申请https证书都是要钱的，这下免费的来了，大家都开始用了，为了实现全网https话，让我们前进吧。

- 环境

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ uname -a
Linux ansheng 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
$ whoami
root
```

我们使用的申请工具是[Certbot](https://certbot.eff.org/)，这里我使用CentOS7+Nginx进行自动签证，如果你想用其他的方式，访问[Certbot](https://certbot.eff.org/)。

# 实操

- 安装epel源

```bash
$ yum install -y epel-release
```

- 安装Certbot

```bash
$ yum install python2-certbot-nginx -y
```

- 安装Nginx

```bash
$ yum install nginx -y
```

我这里用`ssl.ansheng.me`这个域名来做实验，需要在域名管理里面增加一条`A记录`，然后IP只想我们自己服务器的IP，我这里的IP是`149.129.86.210`

```bash
$ ping -c 2 ssl.ansheng.me
PING ssl.ansheng.me (149.129.86.210) 56(84) bytes of data.
64 bytes from 149.129.86.210 (149.129.86.210): icmp_seq=1 ttl=64 time=0.342 ms
64 bytes from 149.129.86.210 (149.129.86.210): icmp_seq=2 ttl=64 time=0.288 ms

--- ssl.ansheng.me ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.288/0.315/0.342/0.027 ms
```

- 对Nginx进行一个简单的配置

```bash
$ vim /etc/nginx/conf.d/ssl.conf
server {
    listen       80;
    server_name  ssl.ansheng.me;
}
```

- 启动nginx

```bash
$ systemctl start nginx
```

- 使用Certbot进行签证

```bash
$ certbot --nginx certonly
......
# 我这里进行签证的时候，会报这个错误，其他服务器都没有这个错误，只有阿里云有，操蛋
ImportError: No module named 'requests.packages.urllib3'
```

在[github](https://github.com/certbot/certbot/issues/5104)找到了解决办法，如下：

```bash
# 这句是我最近加的
$ rm -fr /usr/lib/python2.7/site-packages/urllib3/packages/ssl_match_hostname
$ pip uninstall requests
$ pip uninstall urllib3
$ yum remove python-urllib3
$ yum remove python-requests
$ yum install python-urllib3
$ yum install python-requests
$ yum install python2-certbot-nginx
```

然后再重新申请

```bash
$ certbot --nginx certonly
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): ianshengme@gmail.com   # 输入自己的邮箱，签证完成之后会给你发邮件，然后你自己激活下就成了
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A  # 同意

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y  # Yes
Starting new HTTPS connection (1): supporters.eff.org

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: ssl.ansheng.me
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1  # 输入域名的需要，如果有多个域名，以逗号隔开，类似"1,2,3"
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for ssl.ansheng.me
Waiting for verification...
Cleaning up challenges
Resetting dropped connection: acme-v02.api.letsencrypt.org

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/ssl.ansheng.me/fullchain.pem  # 公钥
   Your key file has been saved at:
   /etc/letsencrypt/live/ssl.ansheng.me/privkey.pem  # 私钥
   Your cert will expire on 2019-04-09. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

- Nginx配置https

```bash
$ vim /etc/nginx/conf.d/ssl.conf
server {
    listen       80;
    server_name  ssl.ansheng.me;

    return 301 https://$server_name$request_uri;
}

server {
    listen       443 ssl http2;
    server_name  ssl.ansheng.me;

    charset utf-8;

    ssl_certificate "/etc/letsencrypt/live/ssl.ansheng.me/fullchain.pem";
    ssl_certificate_key "/etc/letsencrypt/live/ssl.ansheng.me/privkey.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
}
```

上面的配置中，访问`http://ssl.ansheng.me`会强制跳转到`https://ssl.ansheng.me`

- 重新加载配置

```bash
$ nginx -s reload
```

- 通过curl命令进行测试

```bash
$ curl -I http://ssl.ansheng.me
HTTP/1.1 301 Moved Permanently
Server: nginx/1.12.2
Date: Wed, 09 Jan 2019 10:03:07 GMT
Content-Type: text/html
Content-Length: 185
Connection: keep-alive
Location: https://ssl.ansheng.me/  # 跳转正常
```

然后我们在访问`https://ssl.ansheng.me/`

```bash
$ curl -I https://ssl.ansheng.me/
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Wed, 09 Jan 2019 10:04:42 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 3700
Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
Connection: keep-alive
ETag: "5a9e5ebd-e74"
Accept-Ranges: bytes
```

`https`也可以正常访问，这时你可以浏览器打开`https://ssl.ansheng.me/`，然后看看SSL证书的信息。


# 自动续费

在`crontab`里面增加一个定时任务，每天都执行，快到期的时候就会续费了

```bash
$ crontab -l
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew
```