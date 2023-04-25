# Ceph集群增加Mon节点

- 节点信息

|主机名|系统|私有IP|公有IP|
|:--|:--|:--|:--|
|node6|CentOS 8|192.168.200.106|192.168.201.106|

系统初始化可以参考[CentOS手动部署Ceph集群](https://ansheng.me/centos-manual-deployment-of-ceph-cluster/)中的安装部分，我们需要确保以下操作：

1. 设置主机名；
2. 更新hosts将node5的信息增加上去
3. 配置SSH Key
- 修改配置文件

在某台Mon节点上修改`ceph.conf`配置文件增加相应的`mon initial members`与`mon host`

```
$ vi /etc/ceph/ceph.conf
......
mon initial members = node0, node1, node2, node6
mon host = 192.168.200.100, 192.168.200.101, 192.168.200.102, 192.168.200.106
......

```

将配置文件`同步到所有节点`

```
scp /etc/ceph/ceph.conf node1:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node2:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node3:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node4:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node5:/etc/ceph/ceph.conf
scp /etc/ceph/ceph.conf node6:/etc/ceph/ceph.conf

```

在每台节点上面执行以下操作授权

```
chown ceph.ceph /etc/ceph/ceph.conf

```

获取集群已有的mon.keyring

```
$ ceph auth get mon. -o ./mon.keyring

```

获取集群已有的monmap

```
$ ceph mon getmap -o ./monmap

```

查看monmap内容

```
$ monmaptool --print ./monmap

```

将文件scp到node6节点

```
$ scp mon.keyring monmap node6:/tmp/
$ scp /etc/ceph/ceph.client.admin.keyring node6:/etc/ceph/ceph.client.admin.keyring

```

设置相关文件权限

```
$ chown ceph:ceph /var/lib/ceph -R && chown ceph:ceph /etc/ceph -R && chown ceph:ceph /tmp/mon.keyring && chown ceph:ceph /tmp/monmap

```

在新节点上创建监视数据目录并初始化

```
$ sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node6
$ sudo -u ceph ceph-mon --mkfs -i node6 --monmap /tmp/monmap --keyring /tmp/mon.keyring

```

创建done文件，标记mon安装完成

```
$ sudo -u ceph touch /var/lib/ceph/mon/ceph-node6/done

```

启动ceph-mon进程

```
sudo systemctl enable --now ceph-mon@node6
sudo systemctl status ceph-mon@node6

```

随后使用ceph -s命令查看集群状态

```
$ ceph -s
  cluster:
    id:     6ad660a2-ddf1-4c85-a852-4a9f789cdfcd
    health: HEALTH_OK

  services:
    mon: 4 daemons, quorum node0,node1,node2,node6 (age 112s)
    mgr: node0(active, since 22h), standbys: node1, node2
    mds: 1/1 daemons up, 2 standby
    osd: 22 osds: 22 up (since 77m), 22 in (since 80m)

  data:
    volumes: 1/1 healthy
    pools:   3 pools, 41 pgs
    objects: 43 objects, 6.4 KiB
    usage:   202 MiB used, 110 GiB / 110 GiB avail
    pgs:     41 active+clean

```