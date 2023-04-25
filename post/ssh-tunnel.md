# 两条指令搞定SSH反向隧道

## 场景

假设我有两台电脑，一台办公`(A)`的一台家用`(B)`的，因为都是台式机所以呢就很不方便携带，当然了，作为一名开发，用的都是`Linux OS`了，这个时候有可能我需要在电脑`A`上面远程连接到`B`电脑，做一些事情，有些只能在内网做的事情。

> 所以需求就是我需要在家里连接到办公室的电脑

## 环境

|主机|IP|作用|
|:--|:--|:--|
|`A`|172.16.1.10|家里的老式台式机|
|`B`|192.168.1.10|办公室的新式台式机|
|`C`|45.77.131.34|放在公网用来翻墙的一台云主机|

## 原理

![1510281090869766](/images/2017/11/1510281090869766.png)

## 实施

在主机B上建立一个SSH 隧道，将C的6766端口转发到B的22端口上
```bash
$ ssh -p 22 -qngfNTR 6766:localhost:22 root@ss.ansheng.me
```

然后在A主机上利用连接到C主机，然后再SSH连接6766端口反向SSH到B

```bash
$ ssh root@ss.ansheng.me
[root@ansheng ~]# ssh -p 6766 hzhang@localhost
hzhang@Pavilion-dv7:~$ ifconfig
wlp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.181  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::e646:516:f01b:f478  prefixlen 64  scopeid 0x20<link>
        ether 00:16:ea:e0:f5:94  txqueuelen 1000  (Ethernet)
        RX packets 53179  bytes 47259835 (47.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 36976  bytes 5379534 (5.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
完。

## 参考

看起来貌似很简单，自己玩玩还是很不错的，但是你每一次连接上去之后，在退出，隧道就会断开，你可能需要考虑如下情况：

1. 维持稳定的SSH隧道，断开之后自动重建。
2. 系统重启之后如何保持SSH隧道开机启动？

当然了，这些问题在下面的文章中都有讲到，可以参考。

- [使用SSH反向隧道进行内网穿透](http://arondight.me/2016/02/17/使用SSH反向隧道进行内网穿透/)
- [使用SSH反向隧道进行内网穿透](https://www.zhukun.net/archives/8130)
- [ssh隧道反向代理实现内网到公网端口转发](http://www.netcan666.com/2016/09/28/ssh隧道反向代理实现内网到公网端口转发/)
- [内网穿透系列——SSH反向隧道 （最简单的内网穿透方案）](http://www.senra.me/nat-traversal-series-ssh-reverse-tunnel-the-easies-solution/)