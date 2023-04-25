# CentOS 7上安装部署BBR+SS

实验环境如下

```bash
$ uname -a
Linux host.localdomain 3.10.0-514.el7.x86_64 #1 SMP Tue Nov 22 16:42:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ whoami
root
```

习惯性的更新一下系统

```bash
$ yum clean all
$ yum makecache
$ yum update -y
$ shutdown -r now
```

## BBR

BBR (`Bottleneck Bandwidth and RTT`) 是一种新的拥塞控制算法，由Google开发，供Linux内核的TCP协议栈使用，有了BBR算法，Linux服务器可以显著提高吞吐量并减少连接延迟。

### 使用elrepo RPM存储库升级内核

要使用BBR，需要将`CentOS 7`机器的内核升级到`4.x.x`或者更新，使用elrepo RPM存储库轻松完成该操作。

当前的内核为`3.10.x`

```bash
$ uname -r
3.10.0-957.1.3.el7.x86_64
```

- 安装elrepo仓库

```bash
$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
$ rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

- 使用elrepo安装4.9.0内核

```bash
$ yum --enablerepo=elrepo-kernel install kernel-ml -y
```

- 检查是否安装成功

```bash
$ rpm -qa | grep kernel
kernel-tools-3.10.0-957.1.3.el7.x86_64
kernel-tools-libs-3.10.0-957.1.3.el7.x86_64
kernel-3.10.0-957.1.3.el7.x86_64
kernel-ml-4.19.9-1.el7.elrepo.x86_64
kernel-3.10.0-514.el7.x86_64
```

如上的内核版本`kernel-ml-4.19.9-1.el7.elrepo.x86_64`就是我们安装成功的。

- grub2引导启用4.9.0内核

显示grub2菜单中的所有条目

```bash
$ egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux (4.19.9-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux 7 Rescue 766d6a9cbb4ccb4cffa665fbc544c2bb (3.10.0-957.1.3.el7.x86_64)
CentOS Linux (3.10.0-957.1.3.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb) 7 (Core)
```

新内核`4.19.9`位于第一行，因此将默认引导条目设置为0：

```bash
$ grub2-set-default 0
```

接着重启系统

```bash
$ shutdown -r now
```

重启之后通过`uname -r`指令你会发现新内核已经是我们安装的`4.19.9`这个版本

```bash
$ uname -r
4.19.9-1.el7.elrepo.x86_64
```

### 启用BBR

要启用BBR算法，您需要修改sysctl配置，如下所示：

```bash
$ echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
net.core.default_qdisc=fq
$ echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
net.ipv4.tcp_congestion_control=bbr
$ sysctl -p
net.ipv4.neigh.default.base_reachable_time_ms = 600000
net.ipv4.neigh.default.mcast_solicit = 20
net.ipv4.neigh.default.retrans_time_ms = 250
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

- 验证

查看如下命令的返回结果是否与我一直

```bash
$ sysctl net.ipv4.tcp_available_congestion_control
# 输出
net.ipv4.tcp_available_congestion_control = reno cubic bbr
$ sysctl -n net.ipv4.tcp_congestion_control
# 输出
bbr
```

检查内核模块是否已加载

```bash
$ lsmod | grep bbr
# 输出类似于
tcp_bbr                20480  1
```

## SS

请自行了解何为SS（shadowsock）。

- 下载repo文件并安装shadowsocks-libev

```bash
$ wget -P /etc/yum.repos.d/ https://copr.fedoraproject.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
$ yum update
$ yum install shadowsocks-libev
```

如果安装报类似如下错误

```bash
Error: Package: shadowsocks-libev-3.1.3-1.el7.centos.x86_64 (librehat-shadowsocks)
           Requires: libsodium >= 1.0.4
Error: Package: shadowsocks-libev-3.1.3-1.el7.centos.x86_64 (librehat-shadowsocks)
           Requires: mbedtls
```

说明系统没有启用EPEL，那么我们需要首先启用EPEL，再安装shadowsocks-libev

```bash
$ yum remove epel-release -y
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ yum install -y shadowsocks-libev
```

修改配置文件

```bash
$ vi /etc/shadowsocks-libev/config.json
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "password": "blog.ansheng.me",
    "timeout": 60,
    "method": "aes-256-cfb"
}
```

启动并设置开机自启动

```bash
$ systemctl enable --now shadowsocks-libev
```

查看运行状态

```bash
$ systemctl status shadowsocks-libev
```

### 使用Docker运行

请确保服务器已经安装了`docker`

```bash
$ docker run --name=ss --restart=always -e METHOD=<method> -e PASSWORD=<password> -p<server-port>:8388 -p<server-port>:8388/udp -d shadowsocks/shadowsocks-libev
```