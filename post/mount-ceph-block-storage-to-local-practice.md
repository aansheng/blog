# Ceph块存储挂载到本地实践

在管理节点上创建一个名为rbd的pool

```
$ ceph osd pool create rbd

```

在管理节点上使用rbd命令初始化pool以供RBD使用

```
$ rbd pool init rbd

```

创建一个块设备映像

```
$ rbd create foo --size 4096 --image-feature layering -m 192.168.200.101 -k /etc/ceph/ceph.client.admin.keyring -p rbd

```

将映像映射到块设备

```
$ rbd map foo --name client.admin -m 192.168.200.101 -k /etc/ceph/ceph.client.admin.keyring -p rbd

```

格式化该块设备

```
$ mkfs.ext4 -m0 /dev/rbd/rbd/foo

```

挂载文件系统

```
$ mkdir /mnt/ceph-block-device
$ mount /dev/rbd/rbd/foo /mnt/ceph-block-device

```

将如下内容写入`/etc/fstab`，开机自动挂载

```
$ vim /etc/fstab
/dev/rbd/rbd/foo /mnt/ceph-block-device ext4 defaults        0 0

```

查看挂载点详情

```
$ df -h | grep mnt
/dev/rbd0                    3.9G   16M  3.9G   1% /mnt/ceph-block-device

```

文件写入测试

```bash
$ cd /mnt/ceph-block-device
$ echo ping > pong
$ cat pong
ping
```