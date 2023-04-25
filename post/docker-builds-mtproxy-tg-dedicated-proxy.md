# Docker自建MTProxy TG专用代理

通常在使用TG的时候，都需要开一个代理(Shadowsocks/V2ray/Trojan)才能够连接到Telegram，但有时开代理会出现一些问题，比如国内的一些App检测到代理就禁止访问。

这时候我们可以使用MTProto，MTProto是Telegram官方开发的代理协议，只能由Telegram程序使用，我们只需要在TG程序中配置好MTProto就可以访问tg，且不需要开代理程序。

## 在MTProxy Admin Bot创建proxy

关注这个TG BOT [https://t.me/MTProxybot](https://t.me/MTProxybot)，然后按照下图的方式进行注册

![Untitled](/images/2022/10/20/1.png)

## Docker运行MTProxy

记得把`MTP_PORT`、`MTP_SECRET`、`MTP_TAG`替换成自己的参数。

```bash
$ mkdir -p ~/deploy/mtproto
$ vim ~/deploy/mtproto/docker-compose.yml
version: "3.9"

services:

  mtproto:
    image: seriyps/mtproto-proxy
    container_name: mtproto
    restart: always
    network_mode: host
    environment:
      - MTP_PORT=10000
      - MTP_SECRET=47e607ea20cc04ec34074455cfbccd5f
      - MTP_TAG=78f84f516be8f6d4161453e3950da427
      - MTP_DD_ONLY=t
      - MTP_TLS_ONLY=t
$ cd ~/deploy/mtproto
$ docker-compose up -d
```

查看代理链接

```bash
$ docker-compose logs -f
mtproto  | Exec: /opt/mtp_proxy/erts-10.3.5.19/bin/erlexec -noinput +Bd -boot /opt/mtp_proxy/releases/0.1.0/mtp_proxy -mode embedded -boot_var SYSTEM_LIB_DIR /opt/mtp_proxy/lib -config /opt/mtp_proxy/releases/0.1.0/sys.config -args_file /opt/mtp_proxy/releases/0.1.0/vm.args -- foreground -mtproto_proxy allowed_protocols [mtp_fake_tls,mtp_secure] -mtproto_proxy ports [#{name => mtproto_proxy, port => 10000, secret => <<"47e607ea20cc04ec34074455cfbccd5f">>, tag => <<"78f84f516be8f6d4161453e3950da427">>}]
mtproto  | Root: /opt/mtp_proxy
mtproto  | /opt/mtp_proxy
mtproto  | +++++++++++++++++++++++++++++++++++++++
mtproto  | Erlang MTProto proxy by @seriyps https://github.com/seriyps/mtproto_proxy
mtproto  | Sponsored by and powers @socksy_bot
mtproto  |
mtproto  | Proxy started on 0.0.0.0:10000 with secret: 47e607ea20cc04ec34074455cfbccd5f, tag: 78f84f516be8f6d4161453e3950da427
mtproto  | Links:
mtproto  | https://t.me/proxy?server=xxx.xxx.xxx.xxx&port=10000&secret=ee47e607ea20cc04ec34074455cfbccd5f73332e616d617a6f6e6177732e636f6d
mtproto  | https://t.me/proxy?server=xxx.xxx.xxx.xxxport=10000&secret=dd47e607ea20cc04ec34074455cfbccd5f
```

其中代理链接是

```
https://t.me/proxy?server=xxx.xxx.xxx.xxxport=10000&secret=dd47e607ea20cc04ec34074455cfbccd5f
```

将这个链接发送到TG的一个对话框中，然后点击链接添加代理

![Untitled](/images/2022/10/20/2.png)

启用之后可以查看连接状态

![Untitled](/images/2022/10/20/3.png)