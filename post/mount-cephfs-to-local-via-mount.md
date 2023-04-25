# 通过mount将cephfs挂载到本地

创建名字为cephfs_data的数据池 pg大小为8

```
$ ceph osd pool create cephfs_data 8

```

创建名字为cephfs_metadata的存储池 pg大小为8

```
$ ceph osd pool create cephfs_metadata 8

```

创建fs

```
$ ceph fs new cephfs cephfs_metadata cephfs_data
new fs with metadata pool 3 and data pool 2

```

查看和统计

```
$ ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]

```

- 挂载文件系统

查看是否支持内核的方式挂载

```
$ stat /sbin/mount.ceph

```

创建挂载目录

```
$ mkdir /mnt/cephfs

```

如果为启动认证通过以下指令挂载

```
$ mount -t ceph 192.168.200.100:6789:/ /mnt/cephfs
# /etc/fstab自动挂载
# 192.168.200.100:6789:/     /mnt/cephfs    ceph   noatime,_netdev    0 2

```

已启用认证，则需要设置用户名和secret，在mon节点上查看认证信息

```
$ cat /etc/ceph/ceph.client.admin.keyring
[client.admin]
	key = AQBDpXpgtBDYIhAALUmHNY1TI4TsUySf0+2CAA==
    ......
$ mount -t ceph 192.168.200.100:6789:/ /mnt/cephfs -o name=admin,secret=AQBDpXpgtBDYIhAALUmHNY1TI4TsUySf0+2CAA==
# /etc/fstab自动挂载
# 192.168.200.100:6789:/     /mnt/cephfs    ceph    name=admin,secret=AQBDpXpgtBDYIhAALUmHNY1TI4TsUySf0+2CAA==,noatime,_netdev    0 2

```

```
$ df -h | grep mnt
192.168.200.100:6789:/        32G     0   32G   0% /mnt/cephfs
$ echo ping > /mnt/cephfs/pong
$ cat /mnt/cephfs/pong
ping

```

检查是否启用cephx认证方法，如果值为none为禁用，cephx为启用

```bash
$ cat /etc/ceph/ceph.conf | grep auth | grep required
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```