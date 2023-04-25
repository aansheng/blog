# 初步认识NFS网络文件系统

## 什么是NFS?

NFS就是`Network FileSystem`的缩写，最早之前是由Sun所发展出来的。他最大的功能就是可以透过网络，让不同的机器、不同的操作系统、可以彼此分享个别的档案(share file)，所以，也可以简单的将他看做是一个file server呢！这个NFS Server可以让你的PC来将网络远程的NFS主机分享的目录，挂载到本地端的机器当中，所以，在本地端的机器看起来，那个远程主机的目录就好象是自己的partition一般！

虽然NFS有属于自己的协议与使用的端口号，但是在资料传送或者其它相关讯息传递的时候，NFS使用的则是一个称为远程过程调用(Remote Procedure Call, RPC)的协议来协助NFS本身的运作！

NFS的主要功能是通过网络让不同的主机系统之间可以彼此共享文件或目录，NFS网络文件系统的使用很像Windows系统的网络共享，安全功能，网络驱动器映射，这也和Linux里面的samba服务类似。

---

## RPC

因为NFS支持的功能很多，而不同的功能都会使用不同的程序来启动，没启动一个功能就会启用一些端口来传输数据，应此，NFS的功能所对应的端口是无法固定的，二十随机取用一些违背使用的端口来作为传输工具，其中Centos5X随机端口小于1024的，而Centos6X都是比较大的。

因为端口不固定，这样一来就会造成NFS客户端与NFS服务端的通讯障碍，运维NFS客户端必须要知道NFS服务器端的数据传输端口才能进行通讯交互数据。

要解决上面的通讯问题困扰，就需要远程调用RPC服务来帮忙了，NFS的RPC服务最主要的功能就是记录每个NFS功能所对应的嘟啊口号，并且在NFS客户端请求时将该端口和功能对应的信息传递给请求数据的NFS客户端，从而可以确保客户端可以连接到正确的NFS端口上去，达到实现数据传输交互数据目的，这个RPC服务很类似NFS服务端和NFS客户端之间的一个中介。

NFS的RPC服务在CentOS5.x下名称为portmap，在CentOS6.x下名称为rocbind

![nfs-rpc](/images/2016/12/1483011827504.png)

## NFS在企业中的应用场景

在企业集群架构中，NFS网络文件系统一般被用来存储共享视频、图片、附件等静态资源文件（一般把网站用户上传的文件放到共享里，例如：BBS产品的图片、附件、头像，注意网站BBS程序不要放在共享里），NFS是当前互联网系统架构中最常用的数据存储服务器之一，特别是中小型网络公司应用频率很高。大公司或门户除了使用NFS之外，还可能会使用MFS、GFS、FASTFS、TFS等分布式文件系统。

## NFS的安装

### nfs-utils

nfs-utils就是提供rpc.nfsd及rpc.mountd这两个NFS daemons与其它相关documents与说明文件、执行档等的套件！这个就是 NFS 的主要套件

### rpcbind 

rpcbind就如同刚刚提的到，我们的NFS其实可以被视为一个RPC server program，而要激活任何一个RPC server program之前，我们都需要做好port的对应(mapping)的工作才行，这个工作其实就是『rpcbind』这个服务所负责的！也就是说，在激活任何一个RPC server之前，我们都需要激活rpcbind才行呢！那么这个rpcbind到底在干嘛呢？就如同这个服务的名称，哈哈！就是作port的mapping啊！举个例子来说：当Client端尝试来使用RPC server所提供的服务时，由于Client需要取得一个可以连接的port才能够使用RPC server所提供的服务，因此，Client首先就会去跟rpcbind讲『喂！可不可以通知一下，给我个port number，好让我可以跟RPC联络吧！』这个时候rpcbind就自动的将自己管理的port mapping告知Client，好让他可以连接上来server呢！『激活NFS之前，请先激活rpcbind』

### 查询有没有安装软件包

```bash
[root@ansheng ~]# rpm -qa | grep -E "nfs|rpcbind"
```

### 安装软件包

安装包

```bash
[root@ansheng ~]# yum -y install nfs-utils rpcbind
```

或者安装组

```bash
[root@ansheng ~]# yum -y groupinstall "NFS file server"
```

查询

```bash
[root@ansheng ~]# rpm -qa | grep -E "nfs|rpcbind"
nfs-utils-1.2.3-64.el6.x86_64
nfs-utils-lib-1.1.5-11.el6.x86_64
nfs4-acl-tools-0.3.3-7.el6.x86_64
rpcbind-0.2.0-11.el6_7.x86_64
```

## NFS配置文件路径

|NFS常用路径|说明|
|:--|:--|
|/etc/exports|NFS服务主配置文件，配置NFS具体共享服务的地点，默认内容为空|
|/usr/sbin/exportfs|NFS的管理命令，见第五节介绍|
|/usr/sbin/showmount|常用来在客户端，查看NFS配置及共享目录的情况|
|/var/lib/nfs/etab|NFS配置文件的完整参数设定的文件（有很多没有配置单默认就有的参数）|

## showmount命令

**语法：**
 showmount [-ae] hostname

**参数：**

|参数|说明|
|:--|:--|
|-a|显示目前主机与client所连上来的使用目录的状态|
|-e|显示hostname的/etc/exports里面共享的目录|

**实例：**

查看NFS服务端共享的目录

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
/mnt 172.16.10.0/24
```

## exportfs命令

如果我们修改了/etc/exports后，并不需要重启nfs服务，只要用exportfs重新扫描一次/etc/exports，并且重新加载即可。

**语法:**
 exportfs [-aruv]

**参数：**

|参数|说明|
|:--|:--|
|-a|全部挂载(或卸载) /etc/exports档案内的设定|
|-r|重新挂载/etc/exports里面的设定|
|-u|卸载某一目录|
|-v|在export的时候，将分享的目录显示到荧屏上|

**实例：**

exportfs不重启生效法

启动NFS服务

```bash
[root@ansheng ~]# /etc/init.d/rpcbind start
Starting rpcbind:                                          [  OK  ]
[root@ansheng ~]# /etc/init.d/nfs start
Starting NFS services:                                     [  OK  ]
Starting NFS quotas:                                       [  OK  ]
Starting NFS mountd:                                       [  OK  ]
Starting NFS daemon:                                       [  OK  ]
Starting RPC idmapd:                                       [  OK  ]
```

查看是否有共享的目录

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
```

添加共享目录

```bash
[root@ansheng ~]# echo '/mnt 172.16.10.0/24(rw,sync)' >> /etc/exports
[root@ansheng ~]# cat /etc/exports 
/mnt 172.16.10.0/24(rw,sync)
```

不重启NFS服务器查看是否有共享的目录

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
```

利用exportfs生效NFS配置文件

```bash
[root@ansheng ~]# exportfs -rv
exporting 172.16.10.0/24:/mnt
```

查看共享目录

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
/mnt 172.16.10.0/24
```

exportfs卸载所有的共享目录

```bash
[root@ansheng ~]# exportfs -au
```

查看共享目录

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
```

## NFS server端设置

### /etc/exports

共享的目录 主机名称1或IP1(参数1，参数2) 主机名称2或IP2(参数3，参数4)

```bash
	EXAMPLE
       # sample /etc/exports file
       /               master(rw) trusty(rw,no_root_squash)
       /projects       proj*.local.domain(rw)
       /usr            *.local.domain(ro) @trusted(rw)
       /home/joe       pc001(rw,all_squash,anonuid=150,anongid=100)
       /pub            *(ro,insecure,all_squash)
       /srv/www        -sync,rw server @trusted @external(ro)
       /foo            2001:db8:9:e54::/64(rw) 192.0.2.0/24(rw)
       /build          buildhost[0-9].local.domain(rw)
```

### NFS配置参数

|参数|含义|
|:--|:--|
|rw|Read-Write,表示可读写的权限|
|ro|Read-only,表示只读的权限|
|no_root_squash|登入NFS主机使用分享目录的使用者，如果是root的话，那么对于这个分享的目录来说，他就具有root的权限！这个项目『极不安全』，不建议使用！|
|root_squash|在登入NFS主机使用分享之目录的使用者如果是root时，那么这个使用者的权限将被压缩成为匿名使用者，通常他的UID与GID都会变成nobody或者nfsnobody身份；|
|all_squash|不论登入NFS的使用者身份为何，他的身份都会被压缩成为匿名使用者，同时他的UID与GID都会变成nobody或nfsnobody账号身份，在多个NFS客户端同时读写NFS server数据时，这个参数很有用；|
|anonuid|前面关于*_squash提到的匿名使用者的UID设定值，通常为nobody，但是你可以自行设定这个UID的值！当然，这个UID必需要存在于你的/etc/passwd当中！|
|anongid|同anonuid，但是变成group ID就是了|
|sync|请求或写入数据时，数据同步写入到NFS Server的硬盘后才返回。##数据不会丢。缺点，性能差|
|async|请求或写入数据时，先返回请求，再将数据写入到内存缓存和硬盘中，即异步写入数据。此参数可以提升NFS性能，但会降低数据的阿泉。因此，一般情况下不建议使用，如果NFS处于瓶颈状态，并且允许数据丢失的话可以打开此参数提升NFS性能。写入时会先放到内存缓冲区，等硬盘有空档在写入磁盘，这样可以提升写入效率，风险为若服务器宕机或不正常关机，会损失缓冲区中未写入磁盘的数据（服务器主板电视，UPS电源）！|
|Secure|不允许客户端使用大于1024的客户端port，也就是从服务端传递资料到客户端的目的地要小于1024，此时客户端一定要使用root账号才能mount远端NFS server。通常会建议使用。|

## 客户端IP地址

|客户端地址|具体地址例子|说明|
|:--|:--|:--|
|授权单一客户访问NFS|10.0.0.30|一般情况，生产环境中此配置不多|
|授权整个网段可访问NFS|10.0.0.0/24|其中的24等于255.255.255.0，制定瓦工段为生产环境中最常见的配置。配置简单，维护方便|
|授权整个网段可访问NFS|10.0.0.*|指定网段的另外写法（需要验证）|
|授权某个域名客户端访问NFS|Nfs.oldboy.cc|此方法生产环境中一般情况不常用|
|授权某个域名客户端访问NFS|*.oldboy.cc|此方法生产环境中一般情况不常用|


### 配置NFS生产重要技巧

1. 确保所有服务器对NFS共享目录具备相同的权限。
1.1 all_squash把所有客户端都压缩成匿名用户
1.2 就是anonuid,anongid制定的UID和GID的用户。
2. 所有的客户端和服务端都需要有一个相同的UID和GID的用户，即nfsnodoby(UID必须相同)，如果客户端没有服务端指定的用户则创建文件时客户端会显示nfsnodoby这个用户，服务端则显示自己设置的用户；


## 实战配置NFS服务

### 配置NFS服务端

安装NFS软件

```bash
yum –y install nfs-utils rpcbind
```

配置NFS服务

```bash
[root@ansheng ~]# echo "/mnt 172.16.10.0/24(rw,sync)" >> /etc/exports
[root@ansheng ~]# cat /etc/exports 
/mnt 172.16.10.0/24(rw,sync)
```

设置授权

```bash
[root@ansheng ~]# chown -R nfsnobody.nfsnobody /mnt/ 
```

> 查看NFS默认使用的用户以及共享的参数：`cat /var/lib/nfs/etab`

启动服务并设置开机启动

```bash
[root@ansheng ~]# /etc/init.d/rpcbind start
Starting rpcbind:                                          [  OK  ]
[root@ansheng ~]# /etc/init.d/nfs start
Starting NFS services:                                     [  OK  ]
Starting NFS quotas:                                       [  OK  ]
Starting NFS mountd:                                       [  OK  ]
Starting NFS daemon:                                       [  OK  ]
Starting RPC idmapd:                                       [  OK  ]
[root@ansheng ~]# chkconfig rpcbind on
[root@ansheng ~]# chkconfig nfs on
```

查看rpcbind与nfs服务状态

```bash
[root@ansheng ~]# /etc/init.d/rpcbind status
rpcbind (pid  2226) is running...
[root@ansheng ~]# /etc/init.d/nfs status
rpc.svcgssd is stopped
rpc.mountd (pid 2269) is running...
nfsd (pid 2285 2284 2283 2282 2281 2280 2279 2278) is running...
rpc.rquotad (pid 2264) is running...
```

重新加载服务（优雅重启）

```bash
[root@ansheng ~]# /etc/init.d/nfs reload
```

或者

```bash
[root@ansheng ~]# exportfs -r
```

检查或测试挂载

```bash
[root@ansheng ~]# showmount -e localhost
Export list for localhost:
/mnt 172.16.10.0/24
```

本地挂载

```bash
[root@ansheng ~]# mount 172.16.10.10:/mnt /nfstest/         
[root@ansheng ~]# df -h
Filesystem         Size  Used Avail Use% Mounted on
/dev/sda3           19G  2.6G   15G  15% /
tmpfs              932M     0  932M   0% /dev/shm
/dev/sda1          190M   73M  108M  41% /boot
172.16.10.10:/mnt   19G  2.6G   15G  15% /nfstest
[root@ansheng ~]# ls /nfstest/
testnsf.txt
[root@ansheng ~]# cat /nfstest/testnsf.txt 
hello word!
```

## NFS客户端挂载

安装软件

```bash
[root@nfs-node01 ~]# yum -y install rpcbind
```

启动RPCbind设置开机自启动

```bas
[root@nfs-node01 ~]# /etc/init.d/rpcbind start
Starting rpcbind:                                          [  OK  ]
[root@nfs-node01 ~]# chkconfig rpcbind on
```


**查看服务端共享的目录**

安装showmount命令

```bash
[root@nfs-node01 ~]# yum -y install nfs-utils
```

查看共享目录

```bash
[root@nfs-node01 ~]# showmount -e 172.16.10.10
Export list for 172.16.10.10:
/mnt 172.16.10.0/24
```

挂载到本地

```bash
[root@nfs-node01 ~]# mkdir /nfs
[root@nfs-node01 ~]# mount -t nfs 172.16.10.10:/mnt /nfs
[root@nfs-node01 ~]# df -h
Filesystem         Size  Used Avail Use% Mounted on
/dev/sda3           19G  3.4G   15G  20% /
tmpfs              932M     0  932M   0% /dev/shm
/dev/sda1          190M   73M  108M  41% /boot
172.16.10.10:/mnt   19G  2.6G   15G  15% /nfs
[root@nfs-node01 ~]# cat /nfs/testnsf.txt 
hello word!
```

卸载已经挂载的NFS

```bash
[root@nfs-node01 ~]# umount /nfs
[root@nfs-node01 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        19G  3.4G   15G  20% /
tmpfs           932M     0  932M   0% /dev/shm
/dev/sda1       190M   73M  108M  41% /boot
```

**强制卸载：** `umount -lf`

## 开机自动挂载NFS

把挂载的命令写入到/etc/rc.local里面，命令写全路径，这样在启动系统的时候会读取这个文件并且执行挂载的命令，如果写入fstab是不能够实现开机自动挂载的，因为在开机的启动网卡之前会先读取/etc/fstab这个文件，当读取到这个文件系统的时候，网卡并没有启动，这样我要挂载网络文件系统是挂不上的。

```bash
[root@nfs-node01 ~]# echo '/bin/mount -t nfs 172.16.10.10:/mnt/ /nfs' >> /etc/rc.local
[root@nfs-node01 ~]# tail -1 /etc/rc.local 
/bin/mount -t nfs 172.16.10.10:/mnt/ /nfs
```