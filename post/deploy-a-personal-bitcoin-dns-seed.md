# 部署个人的比特币DNS Seed

## 什么是DNS Seed？

通常我们希望在运行一个bitcoin节点时，程序可以实现节点在启动的时候自动发现活跃的节点信息，自动进行连接和同步。

bitcoin源代码中，会在程序内硬编码几个DNS域名到代码中，节点会从默认程序配置的种子节点获取获取其他bitcoin节点，以进行数据同步。

[bitcoin-seeder](https://github.com/sipa/bitcoin-seeder)是社区开发并进行维护的，下面我们将部署将使用bitcoin-seeder。

## 编译

首先我们需要运行一个Docker容器

```bash
docker run --name dnsc --hostname dnsc -d -it --net host debian:11
or
docker run --name dnsc --hostname dnsc -d -it -p 53:53/udp debian:11
```

因为bitcoin-seeder会用到53端口，我们需要让防火墙把53端口允许访问

```bash
ufw allow 53/udp
```

进入容器

```bash
docker exec -it dnsc bash
```

安装所需要的软件包

```bash
apt update && apt install build-essential libboost-all-dev libssl-dev dnsutils git net-tools -y
```

下载源码

```bash
git clone https://github.com/sipa/bitcoin-seeder.git
```

编译程序

```bash
cd bitcoin-seeder
make
```

## 运行

创建工作目录

```bash
mkdir -p ~/deploy/dnsc
mv dnsseed ~/deploy/dnsc/
cd ~/deploy/dnsc/
```

通过以下的指令运行，此时窗口不要关闭，等待采集节点信息

```bash
./dnsseed --testnet -h localhost -n localhost -m info@ansheng.me
```

查看监听的端口

```bash
$ netstat -tulnp | grep 53
udp6       0      0 :::53                   :::*                                6033/./dnsseed
```

大概等两分钟之后会有下面的输出，已经扫描出有1321个节点是活跃可用的

```
[22-07-25 07:31:01] 33/2834 available (128 tried in 54s, 1385 new, 1321 active), 0 banned; 0 DNS requests, 0 db queries
```

## 测试

我们通过dig进行测试

```bash
$ dig localhost @localhost

; <<>> DiG 9.16.27-Debian <<>> localhost @localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42050
;; flags: qr aa rd ad; QUERY: 1, ANSWER: 26, AUTHORITY: 1, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;localhost.			IN	A

;; ANSWER SECTION:
localhost.		3600	IN	A	37.187.130.92
localhost.		3600	IN	A	2.86.29.42
localhost.		3600	IN	A	52.213.124.148
localhost.		3600	IN	A	198.200.30.38
localhost.		3600	IN	A	109.233.61.10
localhost.		3600	IN	A	5.9.5.135
localhost.		3600	IN	A	51.91.221.142
localhost.		3600	IN	A	62.171.128.32
localhost.		3600	IN	A	18.158.238.143
localhost.		3600	IN	A	142.93.23.235
localhost.		3600	IN	A	23.88.49.107
localhost.		3600	IN	A	31.14.40.18
localhost.		3600	IN	A	54.229.31.42
localhost.		3600	IN	A	3.68.67.193
localhost.		3600	IN	A	18.183.173.67
localhost.		3600	IN	A	94.130.108.38
localhost.		3600	IN	A	91.121.146.198
localhost.		3600	IN	A	44.193.18.162
localhost.		3600	IN	A	144.24.252.39
localhost.		3600	IN	A	47.243.91.250
localhost.		3600	IN	A	5.9.121.164
localhost.		3600	IN	A	195.201.95.119
localhost.		3600	IN	A	161.35.142.235
localhost.		3600	IN	A	95.217.203.42
localhost.		3600	IN	A	71.171.85.29
localhost.		3600	IN	A	185.75.76.62

;; AUTHORITY SECTION:
localhost.		40000	IN	NS	localhost.

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon Jul 25 07:48:37 UTC 2022
;; MSG SIZE  rcvd: 466
```

## 发布到互联网

要想将我们的seed发布到互联网，我们需要准备一个域名，比如我的域名是`ansheng.me`，我们需要添加一条A记录和一条NS记录，如下所示

![Untitled](/images/2022/07/25/1.png)

查看NS记录是否生效，这里有一条NS记录`seed.bitcoin.ansheng.me`解析到`ns.bitcoin.ansheng.me`

```bash
$ dig -t NS seed.bitcoin.ansheng.me

; <<>> DiG 9.16.27-Debian <<>> -t NS seed.bitcoin.ansheng.me
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37438
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4e881a311ae6ce46 (echoed)
;; QUESTION SECTION:
;seed.bitcoin.ansheng.me.	IN	NS

;; ANSWER SECTION:
seed.bitcoin.ansheng.me. 4502	IN	NS	ns.bitcoin.ansheng.me.

;; Query time: 369 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Jul 25 07:54:36 UTC 2022
;; MSG SIZE  rcvd: 81
```

查看A记录是否生效，`ns.bitcoin.ansheng.me.`指向DNSc服务器`35.77.82.104`

```bash
$ dig -t A ns.bitcoin.ansheng.me

; <<>> DiG 9.16.27-Debian <<>> -t A ns.bitcoin.ansheng.me
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34662
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 4ee23c19cb8afd8f (echoed)
;; QUESTION SECTION:
;ns.bitcoin.ansheng.me.		IN	A

;; ANSWER SECTION:
ns.bitcoin.ansheng.me.	4502	IN	A	35.77.82.104

;; Query time: 5 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Jul 25 07:55:42 UTC 2022
;; MSG SIZE  rcvd: 78
```

然后通过下面的指令重启dns seed

```bash
$ ./dnsseed --testnet -h seed.bitcoin.ansheng.me -n ns.bitcoin.ansheng.me -m info@ansheng.me
```

等待刚才添加的记录生效后，在其他机器上面通过dig进行测试

```bash
$ dig  seed.bitcoin.ansheng.me

; <<>> DiG 9.16.27-Debian <<>> seed.bitcoin.ansheng.me
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35152
;; flags: qr rd ra; QUERY: 1, ANSWER: 24, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: d399947f019e28ed (echoed)
;; QUESTION SECTION:
;seed.bitcoin.ansheng.me.	IN	A

;; ANSWER SECTION:
seed.bitcoin.ansheng.me. 4502	IN	A	3.126.103.252
seed.bitcoin.ansheng.me. 4502	IN	A	135.148.53.183
seed.bitcoin.ansheng.me. 4502	IN	A	185.186.208.124
seed.bitcoin.ansheng.me. 4502	IN	A	3.68.67.193
seed.bitcoin.ansheng.me. 4502	IN	A	167.179.98.113
seed.bitcoin.ansheng.me. 4502	IN	A	157.90.183.149
seed.bitcoin.ansheng.me. 4502	IN	A	217.26.47.10
seed.bitcoin.ansheng.me. 4502	IN	A	5.2.64.188
seed.bitcoin.ansheng.me. 4502	IN	A	82.221.143.75
seed.bitcoin.ansheng.me. 4502	IN	A	176.223.134.130
seed.bitcoin.ansheng.me. 4502	IN	A	144.24.252.39
seed.bitcoin.ansheng.me. 4502	IN	A	176.9.71.156
seed.bitcoin.ansheng.me. 4502	IN	A	154.12.243.171
seed.bitcoin.ansheng.me. 4502	IN	A	34.89.83.126
seed.bitcoin.ansheng.me. 4502	IN	A	178.18.253.7
seed.bitcoin.ansheng.me. 4502	IN	A	189.123.177.128
seed.bitcoin.ansheng.me. 4502	IN	A	3.131.152.8
seed.bitcoin.ansheng.me. 4502	IN	A	18.176.119.138
seed.bitcoin.ansheng.me. 4502	IN	A	13.229.93.115
seed.bitcoin.ansheng.me. 4502	IN	A	44.238.21.47
seed.bitcoin.ansheng.me. 4502	IN	A	95.217.203.42
seed.bitcoin.ansheng.me. 4502	IN	A	18.140.86.31
seed.bitcoin.ansheng.me. 4502	IN	A	74.220.255.190
seed.bitcoin.ansheng.me. 4502	IN	A	3.91.2.117

;; Query time: 102 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Mon Jul 25 07:57:11 UTC 2022
;; MSG SIZE  rcvd: 448
```