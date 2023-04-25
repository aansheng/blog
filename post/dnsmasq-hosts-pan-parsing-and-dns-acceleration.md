# 使用dnsmasq支持hosts泛解析与DNS加速

最近遇到一个问题，需要在服务器上对域名进行泛解析，比如访问百度的域名统统解析到`6.6.6.6`，然而发现hosts文件根本就不支持类似`*.baidu.com`的这种写法，于是乎就在网上找了下资料，发现可以通过`dnsmasq`来解决这个问题，原理其实就是本机的DNS指向`dnsmasq`服务器，然后`dnsmasq`通过类似`通配符(*)`的方式进行匹配，凡是匹配到`*.baidu.com`的都解析到`6.6.6.6`。

- 环境介绍

```bash
$ uname -a
Linux ansheng 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
$ whoami
root
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

## 安装

安装非常简单，通过`yum`即可

```bash
$ yum install dnsmasq -y
```

## 配置

先把配置文件备份一份

```bash
$ cp /etc/dnsmasq.conf /etc/dnsmasq.conf_bak
```

`dnsmasq`的配置在配置文件中都有详细的说明，你可以通过阅读配置文件的注释更改自己想要的配置，我只是想做泛解析，所以我的配置如下：

```bash
$ vim /etc/dnsmasq.conf
# 严格按照resolv-file文件中的顺序从上到下进行DNS解析, 直到第一个成功解析成功为止
strict-order

# 监听的IP地址
listen-address=127.0.0.1

# 如果监听的不是localhost，把下面两行注视掉
interface=lo
bind-interfaces

# 设置缓存大小
cache-size=10240

# 泛域名解析，访问任何baidu.com域名都会被解析到6.6.6.6
address=/baidu.com/6.6.6.6
```

域名解析默认读取`/etc/hosts`文件到本地域名配置文件（不支持泛域名）

DNS配置默认读取`/etc/resolv.conf`上游DNS配置文件，如果读取不到`/etc/hosts`的地址解析，就会转发给`resolv.conf`进行解析地址

- DNS配置文件

```bash
$ vim /etc/resolv.conf
# 这些都是常用的DNS，可以配置很多
nameserver 127.0.0.1  # 一定要放在第一个
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

- 启动服务

```bash
$ systemctl enable --now dnsmasq
Created symlink from /etc/systemd/system/multi-user.target.wants/dnsmasq.service to /usr/lib/systemd/system/dnsmasq.service.
```

- 查看运行状态

```
$ systemctl status dnsmasq
● dnsmasq.service - DNS caching server.
   Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2018-12-23 09:00:12 UTC; 3s ago
 Main PID: 3844 (dnsmasq)
   CGroup: /system.slice/dnsmasq.service
           └─3844 /usr/sbin/dnsmasq -k

12月 23 09:00:12 ansheng systemd[1]: Started DNS caching server..
12月 23 09:00:12 ansheng dnsmasq[3844]: started, version 2.76 cachesize 10000
12月 23 09:00:12 ansheng dnsmasq[3844]: compile time options: IPv6 GNU-getopt DBus no-i18n IDN DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth no-DNSSEC loop-detect inotify
12月 23 09:00:12 ansheng dnsmasq[3844]: reading /etc/resolv.conf
12月 23 09:00:12 ansheng dnsmasq[3844]: ignoring nameserver 127.0.0.1 - local interface
12月 23 09:00:12 ansheng dnsmasq[3844]: using nameserver 8.8.8.8#53
12月 23 09:00:12 ansheng dnsmasq[3844]: using nameserver 8.8.4.4#53
12月 23 09:00:12 ansheng dnsmasq[3844]: using nameserver 1.1.1.1#53
12月 23 09:00:12 ansheng dnsmasq[3844]: read /etc/hosts - 6 addresses
```

## 测试

```bash
$ ping baidu.com
PING baidu.com (6.6.6.6) 56(84) bytes of data.
^C
--- baidu.com ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1000ms

$ ping www.baidu.com
PING www.baidu.com (6.6.6.6) 56(84) bytes of data.
^C
--- www.baidu.com ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms

$ ping pan.baidu.com
PING pan.baidu.com (6.6.6.6) 56(84) bytes of data.
^C
--- pan.baidu.com ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms
```

由上可以看到，几乎访问任何`baidu.com`的域名都会被解析到`6.6.6.6`，基本上就达到了我们最初的目的。

## 缓存

`dnsmasq`还有一项非常有用的功能就是可以对已经解析过的域名进行缓存，下次在访问这个域名的时候就可以直接返回IP地址，而不再需要经过DNS查询，这对于`扶墙`的来说，其实也算是一点优化，默认已经配置好了，我们只需要来演示下缓存的效果

- 安装dig工具

```bash
$ yum install bind-utils -y
```

- 演示

```bash
$ dig www.centos.com | grep "Query time"
;; Query time: 88 msec
$ dig www.centos.com | grep "Query time"
;; Query time: 0 msec
$ dig www.centos.com | grep "Query time"
;; Query time: 0 msec
$ dig www.centos.com | grep "Query time"
;; Query time: 0 msec
$ dig www.youtube.com | grep "Query time"
;; Query time: 28 msec
$ dig www.youtube.com | grep "Query time"
;; Query time: 0 msec
$ dig www.qq.com | grep "Query time"
;; Query time: 71 msec
$ dig www.qq.com | grep "Query time"
;; Query time: 0 msec
```

看看上面的对比，查询时间缩小了不少倍，可见缓存已经产生作用。
