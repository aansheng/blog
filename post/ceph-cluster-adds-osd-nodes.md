# Ceph集群增加OSD节点

继上篇文章[CentOS手动部署Ceph集群](https://blog.ansheng.me/post/centos-manual-deployment-of-ceph-cluster/)，我们增加一个节点信息，用来专门往Ceph集群中增加一个OSD节点。

## 节点信息

|主机名|系统|私有IP|公有IP|
|:--|:--|:--|:--|
|node5|CentOS 8|192.168.200.105|192.168.201.105|

系统初始化可以参考[CentOS手动部署Ceph集群](https://blog.ansheng.me/post/centos-manual-deployment-of-ceph-cluster/)中的安装部分，我们需要确保以下操作：

1. 设置主机名；
2. 更新hosts将node5的信息增加上去
3. 配置SSH Key

先将配置文件从node0节点上scp到node5节点

```
node0> $ scp /etc/ceph/ceph.conf node5:/etc/ceph/ceph.conf
node0> $ scp /var/lib/ceph/bootstrap-osd/ceph.keyring node5:/var/lib/ceph/bootstrap-osd/

```

设置文件权限

```
node5> $ chown -R ceph.ceph /var/lib/ceph/bootstrap-osd
node5> $ chown -R ceph.ceph /etc/ceph/

```

挂载

```
node5> $ ceph-volume lvm create --data /dev/sdb
...... # 挂载成功之后会提示如下，需要记录下osd ID
--> ceph-volume lvm activate successful for osd ID: 20
--> ceph-volume lvm create successful for: /dev/sdb

```

通过systemctl重新启动osd并且设置开机自启动

```
node5> $ systemctl restart ceph-osd@20
node5> $ systemctl enable ceph-osd@20

```

创建过程可以分为两个阶段（准备和激活）

- 准备OSD

```
$ ceph-volume lvm prepare --data /dev/sdc

```

一旦准备好，激活就需要准备好的OSD ID和OSD FSID，这些可以通过列出当前服务器中的OSD来获得：

```
$ ceph-volume lvm list
......
====== osd.21 ======

  [block]       /dev/ceph-bbe5e4cc-04e6-448a-9008-18105515ebc3/osd-block-85c66d83-4447-46d6-b007-6567707cf810

      block device              /dev/ceph-bbe5e4cc-04e6-448a-9008-18105515ebc3/osd-block-85c66d83-4447-46d6-b007-6567707cf810
      block uuid                fO9rgq-0qgw-4Iuf-oYMI-HFGk-S2ZS-HFFhZ7
      cephx lockbox secret
      cluster fsid              6ad660a2-ddf1-4c85-a852-4a9f789cdfcd
      cluster name              ceph
      crush device class        None
      encrypted                 0
      osd fsid                  85c66d83-4447-46d6-b007-6567707cf810
      osd id                    21
      osdspec affinity
      type                      block
      vdo                       0
      devices                   /dev/sdc

```

- 激活OSD

```
$ ceph-volume lvm activate 21 85c66d83-4447-46d6-b007-6567707cf810

```

通过systemctl重新启动osd并且设置开机自启动

```
node5> $ systemctl restart ceph-osd@21
node5> $ systemctl enable ceph-osd@21

```

- 在node0节点上查看osd是否已经挂载

```
node0> $ ceph osd tree
ID   CLASS  WEIGHT   TYPE NAME       STATUS  REWEIGHT  PRI-AFF
......
-13         0.00980      host node5
 20    hdd  0.00490          osd.20      up   1.00000  1.00000
 21    hdd  0.00490          osd.21      up   1.00000  1.00000

```