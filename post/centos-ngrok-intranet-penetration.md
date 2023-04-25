# 基于CentOS利用ngrok完美进行内网穿透

在开始之前你需要准备`一台公网服务器`，`一个域名`，给域名添加`两条A记录`，`ngrok`和`dev.ngrok	`，并且都指向到公网服务器上。

## 什么是ngrok？

[ngrok](https://github.com/inconshreveable/ngrok)是一个反向代理，用于创建从公共端点到本地运行的Web服务的安全隧道，可以是TCP或者HTTP。

## 环境

|IP|描述|
|:--|:--|
|`103.80.169.213`|服务端，公网|
|`172.17.0.4`|客户端，内网|

- 客户端

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
```

- 服务端

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
```

> 客户端和服务端使用的都是CentOS

## 服务端配置

### 安装

- 安装epel源

```bash
$ yum install -y epel-release
```

- 更新系统

```bash
$ yum update -y
```

- 安装编译需要的工具包

```bash
$ yum groupinstall -y development tool
$ yum install -y perl-ExtUtils-MakeMaker mercurial
$ yum install -y golang
```

- 下载ngrok源码

```bash
$ cd /opt
$ git clone https://github.com/inconshreveable/ngrok.git
```

- 生成自签名证书

```bash
$ cd ngrok
$ export NGROK_DOMAIN="ngrok.ansheng.me"
$ openssl genrsa -out ngrok.ansheng.me.key 2048
$ openssl req -new -x509 -nodes -key ngrok.ansheng.me.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out ngrok.ansheng.me.pem
$ openssl genrsa -out ngrok.ansheng.me.server.key 2048
$ openssl req -new -key ngrok.ansheng.me.server.key -subj "/CN=$NGROK_DOMAIN" -out ngrok.ansheng.me.server.csr
$ openssl x509 -req -in ngrok.ansheng.me.server.csr -CA ngrok.ansheng.me.pem -CAkey ngrok.ansheng.me.key -CAcreateserial -out ngrok.ansheng.me.server.crt -days 5000
```

- 将生成的秘钥复制到编译目录中

```bash
$ \cp ngrok.ansheng.me.server.crt assets/server/tls/snakeoil.crt
$ \cp ngrok.ansheng.me.server.key assets/server/tls/snakeoil.key
$ \cp ngrok.ansheng.me.pem assets/client/tls/ngrokroot.crt
```

- 编译ngrok服务器端

```bash
$ make release-server
```

编译的时候会从`github clone`依赖的库，如果是国内的网络，可能会比较慢，稍等片刻，如果报错就多执行几次。

- 编译不同平台的客户端

编译Linux64位版

```bash
$ make release-client
```

编译Mac版

```bash
$ GOOS=darwin GOARCH=amd64 make release-client
```

编译Windows64位版

```bash
$ GOOS=windows GOARCH=amd64 make release-client
```

编译Windows32位版

```bash
$ GOOS=windows GOARCH=386 make release-client
```

编译完成之后会生成相关的可执行文件，存放在`/opt/ngrok/bin/`目录下，下面列出了一些可执行文件的说明

|文件名|描述|
|:--|:--|
|`/opt/ngrok/bin/ngrokd`|ngrok服务器端|
|`/opt/ngrok/bin/ngrok`|Linux客户端|
|`/opt/ngrok/bin/darwin_amd64/ngrok`|Mac客户端|
|`/opt/ngrok/bin/windows_amd64/ngrok.exe`|Windows64位客户端|
|`/opt/ngrok/bin/windows_386/ngrok.exe`|Windows32位客户端|

### 配置

如果你启动了防火墙，需要添加以下规则

```bash
# 放行自定义端口，例如：TCP 9022（SSHD）、9080（Web）端口
$ firewall-cmd --zone=public --add-port=9022/tcp --permanent
$ firewall-cmd --zone=public --add-port=9080/tcp --permanent
# ngrokd需要的端口，客户端连接的时候指定的端口
$ firewall-cmd --zone=public --add-port=9443/tcp --permanent
```

重新加载防火墙规则

```bash
$ firewall-cmd --reload
```

- 服务器端启动ngrok

监听`9443`端口

```bash
$ /opt/ngrok/bin/ngrokd -tlsKey=/opt/ngrok/ngrok.ansheng.me.server.key -tlsCrt=/opt/ngrok/ngrok.ansheng.me.server.crt -domain=ngrok.ansheng.me -httpAddr=:9080 -tunnelAddr=:9443
```

## 客户端配置

把服务端生成的Linux客户端软件`scp`到本地

```bash
$ mkdir /opt/ngrok
$ cd /opt/ngrok
$ scp root@103.80.169.213:/opt/ngrok/bin/ngrok .
```

添加配置文件

```bash
$ vi ngrok.cfg
server_addr: ngrok.ansheng.me:9443  # 远程地址
trust_host_root_certs: false
tunnels:  # 隧道
    ssh:  # 隧道名称
        remote_port: 9022  # 绑定到远程地址的那个端口
        proto:
            tcp: 22  # 本地的TCP 22端口
    web:
        subdomain: dev  # 域名前缀，dev.ngrok.ansheng.me
        remote_port: 9080
        proto:
            http: 80  # 本地的HTTP 80端口
```

启动客户端

```bash
# $ ./ngrok -config=./ngrok.cfg start web  # 启动web隧道
$ ./ngrok -config=./ngrok.cfg start-all  # 启动所有隧道
```

启动之后会输出下面的信息，`online`表示运行成功

```
Tunnel Status                 online
Version                       1.7/1.7
Forwarding                    http://dev.ngrok.ansheng.me:9080 -> 127.0.0.1:80
Forwarding                    tcp://ngrok.ansheng.me:9022 -> 127.0.0.1:22
Web Interface                 127.0.0.1:4040
# Conn                        5  # 我这已经有连接了
Avg Conn Time                 38339.43ms
```

为NGINX添加页面信息

```bash
$ echo "172.17.0.4"  > /usr/share/nginx/html/index.html
```

## 测试

请确保你已经在客户端开启了ssh和web服务

- web

```bash
$ curl http://dev.ngrok.ansheng.me:9080/
172.17.0.4
```

- ssh

```bash
$ ssh -p9022 root@ngrok.ansheng.me
# 输入密码进行登录
$ ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.4  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:04  txqueuelen 0  (Ethernet)
        RX packets 35612  bytes 48627989 (46.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 24106  bytes 1345146 (1.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 参考文献

1. [Ngrok Documentation](https://ngrok.com/docs)
2. [阿里云主机CentOS 7下编译安装ngrok](http://www.racksam.com/2016/12/18/aliyun-centos7-install-ngrok/)
3. [最新完美教程 使用centos 7 linux 自己搭建 ngrok 实现内网穿透](https://blog.csdn.net/qq_36560161/article/details/79632595)
4. [Centos7搭建Ngrok](https://blog.csdn.net/u010887744/article/details/53957683)
