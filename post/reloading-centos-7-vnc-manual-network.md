# 通过VNC手动网络重装 CentOS 7

如果要手动重装VPS，请确保你具备有以下条件

1. VPS当前系统是 CentOS 7（因为要用到 grub2）
2. 可以连接 VNC （没有 VNC 就无法使用安装界面）
3. VPS架构只支持KVM不支持OVZ

## 获取网络信息

通过`ifconfig`获取IP地址和子网掩码信息

```bash
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 107.182.30.100（IP地址）  netmask 255.255.240.0（子网掩码）  broadcast 107.182.31.255
        inet6 fe80::a8aa:ff:fe11:f79e  prefixlen 64  scopeid 0x20<link>
        ether aa:aa:00:11:f7:9e  txqueuelen 1000  (Ethernet)
        RX packets 133  bytes 13815 (13.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 106  bytes 13375 (13.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 68  bytes 6264 (6.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68  bytes 6264 (6.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

查询网关地址

```bash
$ route -n
Kernel IP routing table
Destination     Gateway                Genmask         Flags Metric Ref    Use Iface
0.0.0.0         107.182.16.1（网关）    0.0.0.0         UG    100    0        0 eth0
107.182.16.0    0.0.0.0                255.255.240.0   U     100    0        0 eth0
```

以上我们得到了以下信息：

- IP 地址：107.182.30.100
- 子网掩码：255.255.240.0
- 网关地址：107.182.16.1

## 配置启动文件

先安装`wget`命令

```bash
yum install wget -y
```

然后我们下载需要用于网络启动的内核

```bash
wget -O /boot/initrd.img http://mirror.centos.org/centos/7/os/x86_64/images/pxeboot/initrd.img
wget -O /boot/vmlinuz http://mirror.centos.org/centos/7/os/x86_64/images/pxeboot/vmlinuz
cp /boot/initrd.img /
cp /boot/vmlinuz /
```

编辑`/etc/grub.d/40_custom`文件

```bash
$ vi /etc/grub.d/40_custom
...
# 在最后增加以下信息
menuentry "Network Install CentOS 7" {
    set root='(hd0,msdos1)'
    # linux /vmlinuz repo=http://mirror.centos.org/centos/7/os/x86_64/ ip=IP地址 netmask=子网掩码 gateway=网关地址 nameserver=DNS地址
    linux /vmlinuz repo=http://mirror.centos.org/centos/7/os/x86_64/ ip=107.182.30.100 netmask=255.255.240.0 gateway=107.182.16.1 nameserver=1.1.1.1
    initrd /initrd.img
}
```

生成grub配置文件

```bash
grub2-mkconfig --output=/boot/grub2/grub.cfg
```

查看启动项

```
$ egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
CentOS Linux (3.10.0-957.5.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-16fe1aa10c0b925a57abe21439573c6b) 7 (Core)
CentOS Linux (0-rescue-bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb) 7 (Core)
menuentry "Network Install CentOS 7" {
```

因为`Network Install CentOS 7`位于第一行，因此将默认引导条目设置为4

```bash
grub2-set-default 4
```

## 开始安装

当以上操作都完成后，使用`reboot`重启系统，请在重启之前提前打开VNC，准备进行安装

```bash
reboot
```

当进入启动画面后，选择`Network Install CentOS 7`这个菜单

![select-centos](/images/2019/3/23/10df49ae50ccee4648848cd599724862.png)

耐心等待几分钟，就可以看到`CentOS 7`的网络安装器界面了

![install-centos](/images/2019/3/23/7afee312baf93dd93bd2cbecc587dfe2.png)