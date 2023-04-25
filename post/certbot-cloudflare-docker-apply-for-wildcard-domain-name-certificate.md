# CertBot+CloudFlare+Docker申请通配符域名证书

之前有讲到过[Certbot使用DNS和文件验证两种方式申请SSL证书](https://blog.ansheng.me/post/certbot-uses-dns-and-file-verification-to-apply-for-ssl-certificates)，但是这种方式无法申请通配符域名，通配符域名只能通过DNS验证的方式来进行。

CertBot提供了很多[DNS插件](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins)，我们会使用[certbot-dns-cloudflare](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)这个插件来进行自动申请和自动部署。

## 申请CloudFlare API KEY

用DNS方式在申请和续订的时候，都需要添加一条TXT记录来证明对域名的拥有权，但是每次手动添加又会很麻烦，所以我们可以通过调用域名提供商的API，来进行自动添加TXT记录，这样就取消了每次手动添加的麻烦。

- 获取API Key

登录CF账号，点击右上角头像下面的`我的个人资料`

![Untitled](/images/2022/10/18/1.png)

找到左侧`API令牌`—>点击`创建令牌`

![Untitled](/images/2022/10/18/2.png)

因为我们只需要修改DNS的权限，所以选择使用`编辑区域DNS`模版

![Untitled](/images/2022/10/18/3.png)

下面选择本次需要申请证书的域名，为了安全起见，一个API KEY对应一个域名

![Untitled](/images/2022/10/18/4.png)

最后创建令牌

![Untitled](/images/2022/10/18/5.png)

记得保存好API KEY，只会显示一次，后面只能更改了

![Untitled](/images/2022/10/18/6.png)

## 申请证书

创建工作目录

```bash
mkdir -p $HOME/deploy/certbot/data/etc/letsencrypt && cd ~/deploy/certbot
```

将API KEY保存到`cloudflare.ini`文件中

```bash
$ vim data/etc/letsencrypt/cloudflare.ini
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = _APIKEY_
```

安全起见把权限改为600

```bash
chmod 600 data/etc/letsencrypt/cloudflare.ini
```

创建申请脚本

```bash
$ vim apply.sh
#!/bin/bash

docker run --network host -it --rm --name certbot \
  -v "$HOME/deploy/certbot/data/etc/letsencrypt:/etc/letsencrypt" \
  -v "$HOME/deploy/certbot/data/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/dns-cloudflare certonly \
  --agree-tos \
  --email info@ansheng.me \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  -d ansheng.me,*.ansheng.me
```

申请证书

```bash
$ bash apply.sh
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y # 同意
Account registered.
Requesting a certificate for ansheng.me and *.ansheng.me
Waiting 10 seconds for DNS changes to propagate

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ansheng.me/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ansheng.me/privkey.pem
This certificate expires on 2023-01-16.
These files will be updated when the certificate renews.

NEXT STEPS:
- The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.
We were unable to subscribe you the EFF mailing list because your e-mail address appears to be invalid. You can try again later by visiting https://act.eff.org.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

查看证书信息

```bash
$ docker run --network host -it --rm --name certbot \
    -v "$HOME/deploy/certbot/data/etc/letsencrypt:/etc/letsencrypt" \
    -v "$HOME/deploy/certbot/data/var/lib/letsencrypt:/var/lib/letsencrypt" \
    certbot/dns-cloudflare certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: ansheng.me
    Serial Number: xxxxxxxx
    Key Type: RSA
    Domains: ansheng.me *.ansheng.me
    Expiry Date: 2023-01-16 13:40:36+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/ansheng.me/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/ansheng.me/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

NGINX配置好之后的证书信息

![Untitled](/images/2022/10/18/7.png)

## 自动续订

```bash
$ vim renew.sh
#!/bin/bash

docker run --network host -it --rm --name certbot \
  -v "$HOME/deploy/certbot/data/etc/letsencrypt:/etc/letsencrypt" \
  -v "$HOME/deploy/certbot/data/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/dns-cloudflare renew --quiet
```

执行自动续订脚本

```bash
bash renew.sh
```

- 添加到定时任务中每天运行

```bash
$ crontab -e
00 00 * * * /bin/bash ~/deploy/certbot/renew.sh
```