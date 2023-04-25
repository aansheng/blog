# Certbot使用DNS和文件验证两种方式申请SSL证书

[Certbot](https://certbot.eff.org/)是一个申请[Let’s Encrypt](https://letsencrypt.org/zh-cn/)免费SSL的工具，其提供系统安装包以及[docker镜像](https://hub.docker.com/r/certbot/certbot/)的方式运行，本文则通过Docker的方式进行操作。

## DNS

在使用文件验证申请SSL证书时会使用80端口做文件校验，但是有时候我们的域名并非会用到80，而会用到一些非80的端口，这个时候我们使用DNS验证的形式申请证书会更方便一些。

```bash
$ sudo docker run --network host -it --rm --name certbot \\
    -v "/etc/letsencrypt:/etc/letsencrypt" \\
    -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \\
    certbot/certbot certonly -d ssl.ansheng.me --manual --preferred-challenges dns

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): info@ansheng.me # 输入你的邮箱

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
<https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf>. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y # 输入Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y # 输入Y
Account registered.
Requesting a certificate for ssl.ansheng.me

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name: # 需要将以下TXT记录添加DNS中，具体看下图

_acme-challenge.ssl.ansheng.me.

with the following value:

PE4XVQMwkof_r2rZV7SfNQcFWVvb081Mvk0G_u1toj4

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: <https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.ssl.ansheng.me>.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue # 添加之后稍等片刻，然后回车

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ssl.ansheng.me/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ssl.ansheng.me/privkey.pem
This certificate expires on 2022-04-16.
These files will be updated when the certificate renews.

NEXT STEPS:
- This certificate will not be renewed automatically. Autorenewal of --manual certificates requires the use of an authentication hook script (--manual-auth-hook) but one was not provided. To renew this certificate, repeat this same certbot command before the certificate's expiry date.
We were unable to subscribe you the EFF mailing list because your e-mail address appears to be invalid. You can try again later by visiting <https://act.eff.org>.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   <https://letsencrypt.org/donate>
 * Donating to EFF:                    <https://eff.org/donate-le>
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

```

我的域名托管在cloudflare，以下是添加TXT记录的截图

![Untitled](/images/2022/01/16/1.png)

申请成功之后的证书文件存放在`/etc/letsencrypt/live/ssl.ansheng.me/`目录中

```bash
$ ls /etc/letsencrypt/live/ssl.ansheng.me
cert.pem  chain.pem  fullchain.pem  privkey.pem  README
```

## 文件验证

文件验证的形式申请的SSL，首先需要将域名解析到服务器，如果是cloudflare，记得把小云朵关上

![Untitled](/images/2022/01/16/2.png)

我这里用的NGINX，但不管用什么都大同小异，NGINX配置文件如下

```
server {
    listen 80;
    server_name ssl2.ansheng.me;

    location / {
      root   /html/ssl;
      index  index.html index.htm;
    }
}

```

`/html/ssl/`目录中有一个`index.html`文件，内容为：`hello`，可以通过`curl`的方式进行访问测试

```bash
$ curl <http://ssl2.ansheng.me>
hello

```

- 通过CertBOT申请SSL

```bash
$ sudo docker run --network host -it --rm --name certbot \\
    -v "/etc/letsencrypt:/etc/letsencrypt" \\
    -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \\
    -v "/root/deploy/nginx/html/ssl2:/html/ssl2" \\
    certbot/certbot certonly --webroot -w /html/ssl2 -d ssl2.ansheng.me

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for ssl2.ansheng.me

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ssl2.ansheng.me/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ssl2.ansheng.me/privkey.pem
This certificate expires on 2022-04-16.
These files will be updated when the certificate renews.

NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See <https://certbot.org/renewal-setup> for instructions.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   <https://letsencrypt.org/donate>
 * Donating to EFF:                    <https://eff.org/donate-le>
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

```

## 自动续费

默认情况下证书有效期是3个月，可以将以下命令写入`crontab`实现自动续费

```bash
$ sudo docker run --network host -it --rm --name certbot \\
    -v "/etc/letsencrypt:/etc/letsencrypt" \\
    -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \\
    certbot/certbot renew --quiet

```

- `-quiet`参数不输出日志，也可以去掉查看一下日志

## 脚本留存

- crontab

```bash
00 00 * * * /bin/bash ~/deploy/certbot/renew.sh
```

- apply.sh

```bash
#!/bin/bash

docker run --network host -it --rm --name certbot \
    -v "$HOME/deploy/certbot/data/etc/letsencrypt:/etc/letsencrypt" \
    -v "$HOME/deploy/certbot/data/var/lib/letsencrypt:/var/lib/letsencrypt" \
    -v "$HOME/deploy/nginx/html:/usr/share/nginx/html" \
    certbot/certbot certonly --webroot \
    -w /usr/share/nginx/html/dubi -d dubi.eu.org
```

- renew.sh

```bash
#!/bin/bash

docker run --network host -it --rm --name certbot \
  -v "$HOME/deploy/certbot/data/etc/letsencrypt:/etc/letsencrypt" \
  -v "$HOME/deploy/certbot/data/var/lib/letsencrypt:/var/lib/letsencrypt" \
  -v "$HOME/deploy/nginx/html:/usr/share/nginx/html" \
  certbot/certbot renew --quiet
```