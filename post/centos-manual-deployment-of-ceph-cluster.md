# CentOS手动部署Ceph集群

## 节点信息

|主机名|系统|私有IP|公有IP|
|:--|:--|:--|:--|
|node0|CentOS 8|192.168.200.100|192.168.201.100|
|node1|CentOS 8|192.168.200.101|192.168.201.101|
|node2|CentOS 8|192.168.200.102|192.168.201.102|
|node3|CentOS 8|192.168.200.103|192.168.201.103|
|node4|CentOS 8|192.168.200.104|192.168.201.104|

## 安装

安装的部分需要在每个节点上执行同样的操作，整个操作都是用root用户操作

- 系统版本

```
$ cat /etc/redhat-release
CentOS Linux release 8.3.2011

```

- 初始化系统

```
$ dnf install epel-release vim net-tools -y
$ dnf update -y
$ systemctl disable --now firewalld
$ setenforce 0
$ sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config

```

- 配置ceph yum源

yum源是用的pacific版本的ceph

```
$ vim /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-pacific/el8/$basearch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-pacific/el8/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-pacific/el8/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc

```

- 安装ceph

```
$ dnf install snappy leveldb gdisk gperftools-libs -y
$ dnf install ceph -y

```

确保pip安装了以下包

```
$ pip3 install pecan werkzeug

```

- 配置时间同步

```
$ dnf install -y chrony
$ systemctl enable --now chronyd

```

- 增加hosts文件

```
$ cat >> /etc/hosts <<EOF
192.168.200.100 node0
192.168.200.101 node1
192.168.200.102 node2
192.168.200.103 node3
192.168.200.104 node4
EOF

```

- 配置ssh key

在node0上面生成ssh key

```
$ ssh-keygen

```

获取公钥

```
$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZPdLDd8NfUfl9Ke9OBW6S5myv+NrgrgIUp3GdmyNznhacVbT0DXj9UgJxJGCpPCxf2AG2kdJb5vF5MBPMGXxSIFG7lMPOfTc8lNs5yGjQEfZCOM8gjyqxgONCCZ66u9wWiF4rvPZIgIFQM3MO4hrTpS36McU8MjLsYMe4KjEgzqs90rcAz5xNRxizRBii5EHuV6QWNzSTwhxIp0uvGHaFeTej/x7vY/Bxwu5KZg2HUjZWb/E3j+YRQJVw47yqz0jP/GZigdEjm97hMPZ/wb5MwIC1Q1qLr0gHknJ2Qj89f28D5exXSfIrGBiDuzivtsoot6hBsmAogS+0RZZDBTsQNAPd2S2FbKIBQlXpSC2fPyo4qCnORFekQM6oSInxEjc9k1Dc+Bs7c9F72P4PkkqEICa8tD/j3qgoq/egrtYUmGyNolCglDp8XaFd/nYcz15WiNYrkr0Ixjqc8RhezgwR8bN6D6t1+Gm4HgFPsNlKSucE2fbpxPVaLH/+Wp8tUn0= root@node0

```

把公钥分发到node1、node2、node3、node4，以下操作在node1-4上面操作

```
$ mkdir ~/.ssh
$ vim ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZPdLDd8NfUfl9Ke9OBW6S5myv+NrgrgIUp3GdmyNznhacVbT0DXj9UgJxJGCpPCxf2AG2kdJb5vF5MBPMGXxSIFG7lMPOfTc8lNs5yGjQEfZCOM8gjyqxgONCCZ66u9wWiF4rvPZIgIFQM3MO4hrTpS36McU8MjLsYMe4KjEgzqs90rcAz5xNRxizRBii5EHuV6QWNzSTwhxIp0uvGHaFeTej/x7vY/Bxwu5KZg2HUjZWb/E3j+YRQJVw47yqz0jP/GZigdEjm97hMPZ/wb5MwIC1Q1qLr0gHknJ2Qj89f28D5exXSfIrGBiDuzivtsoot6hBsmAogS+0RZZDBTsQNAPd2S2FbKIBQlXpSC2fPyo4qCnORFekQM6oSInxEjc9k1Dc+Bs7c9F72P4PkkqEICa8tD/j3qgoq/egrtYUmGyNolCglDp8XaFd/nYcz15WiNYrkr0Ixjqc8RhezgwR8bN6D6t1+Gm4HgFPsNlKSucE2fbpxPVaLH/+Wp8tUn0= root@node0
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys

```

- 重启系统

```
$ reboot

```

## Mon

以下操作在node0上面操作

- 生成UUID

```
$ uuidgen
6ad660a2-ddf1-4c85-a852-4a9f789cdfcd

```

- 创建ceph配置文件

```
$ vim /etc/ceph/ceph.conf
[global]
# 集群唯一uuid
fsid = 6ad660a2-ddf1-4c85-a852-4a9f789cdfcd

# mon主机名列表，多个用逗号分隔
mon initial members = node0, node1, node2
# mon节点的ip:port列表, 默认ceph-mon服务的端口号是6789
mon host = 192.168.200.100, 192.168.200.101, 192.168.200.102

# 开放客户端访问的网络段
public network = 192.168.200.0/24

# 认证方式
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
osd journal size = 1024

# 默认副本数3
osd pool default size = 3
osd pool default min size = 2

# 配置单个pool默认的pg数量和pgp数量
osd pool default pg num = 333
osd pool default pgp num = 333

# 默认是1表示不允许把数据的不同副本放到1个节点上
osd crush chooseleaf type = 1

# 是否允许删除pool
mon_allow_pool_delete = true

```

将配置文件scp到每台机器上

```
$ scp /etc/ceph/ceph.conf node1:/etc/ceph/ceph.conf
$ scp /etc/ceph/ceph.conf node2:/etc/ceph/ceph.conf
$ scp /etc/ceph/ceph.conf node3:/etc/ceph/ceph.conf
$ scp /etc/ceph/ceph.conf node4:/etc/ceph/ceph.conf

```

创建monitor的key用于多个monitor间通信，保存在/tmp/ceph.mon.keyring

```
$ ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'

```

生成管理用户client.admin及其key，保存在/etc/ceph/ceph.client.admin.keyring

```
$ ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'
# >> key文件命令规则:{cluster name}-{user-name}.keyring

```

生成bootstrap-osd keyring，生成client.bootstrap-osd用户并将用户添加到keyring

```
$ ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd' --cap mgr 'allow r'

```

将生成的密钥添加到中ceph.mon.keyring

```
$ ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
$ ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring

```

改ceph.mon.keyring的权限为ceph

```
$ chown ceph:ceph /tmp/ceph.mon.keyring

```

创建monitor map

```
$ monmaptool --create --add node0 192.168.200.100 --add node1 192.168.200.101 --add node2 192.168.200.102 --fsid 6ad660a2-ddf1-4c85-a852-4a9f789cdfcd /tmp/monmap

```

查看monitor map内容

```
$ monmaptool --print /tmp/monmap

```

创建ceph-mon数据目录

```
node0> $ sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node0
node1> $ sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node1
node2> $ sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node2

```

拷贝以下文件到node1,node2节点

```
$ scp /tmp/monmap node1:/tmp/monmap && scp /tmp/monmap node2:/tmp/monmap
$ scp /tmp/ceph.mon.keyring node1:/tmp/ceph.mon.keyring && scp /tmp/ceph.mon.keyring node2:/tmp/ceph.mon.keyring
$ scp /etc/ceph/ceph.client.admin.keyring node1:/etc/ceph/ceph.client.admin.keyring && scp /etc/ceph/ceph.client.admin.keyring node2:/etc/ceph/ceph.client.admin.keyring

```

三个节点node0,node1,node2设置相关文件权限

```
$ chown ceph:ceph /var/lib/ceph -R && chown ceph:ceph /etc/ceph -R && chown ceph:ceph /tmp/ceph.mon.keyring && chown ceph:ceph /tmp/monmap

```

初始化数据目录

```
node0> sudo -u ceph ceph-mon --mkfs -i node0 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
node1> sudo -u ceph ceph-mon --mkfs -i node1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
node2> sudo -u ceph ceph-mon --mkfs -i node2 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring

```

在每台机器上面操作，创建done文件，标记mon安装完成

```
node0> $ sudo -u ceph touch /var/lib/ceph/mon/ceph-node0/done
node1> $ sudo -u ceph touch /var/lib/ceph/mon/ceph-node1/done
node2> $ sudo -u ceph touch /var/lib/ceph/mon/ceph-node2/done

```

启动ceph-mon进程，启动时候我们需要传递monitor name参数给systemctl，用以指明启动的monitor

```
node0> $ sudo systemctl enable --now ceph-mon@node0
node0> $ sudo systemctl status ceph-mon@node0

node1> $ sudo systemctl enable --now ceph-mon@node1
node1> $ sudo systemctl status ceph-mon@node1

node2> $ sudo systemctl enable --now ceph-mon@node2
node2> $ sudo systemctl status ceph-mon@node2

```

检查端口是否运行

```
$ netstat -tulnp | grep 6789

```

查看启动日志

```
$ tail -f /var/log/ceph/ceph-mon.node0.log

```

查看运行状态

```
$ ceph -s

```

如果有`3 monitors have not enabled msgr2`的WARN提示没有启用msgr2，通过以下指令启动

```
node0> $ sudo -u ceph ceph mon enable-msgr2
node1> $ sudo -u ceph ceph mon enable-msgr2
node2> $ sudo -u ceph ceph mon enable-msgr2

```

## mgr

创建mgr密钥目录

```
node0> $ sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node0
node1> $ sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node1
node2> $ sudo -u ceph mkdir /var/lib/ceph/mgr/ceph-node2

```

创建mgr身份验证密钥，注意里面的mgr.node1，node1为主机名

```
node0> $ sudo -u ceph ceph auth get-or-create mgr.node0 mon 'allow profile mgr' osd 'allow *' mds 'allow *' > /var/lib/ceph/mgr/ceph-node0/keyring
node1> $ sudo -u ceph ceph auth get-or-create mgr.node1 mon 'allow profile mgr' osd 'allow *' mds 'allow *' > /var/lib/ceph/mgr/ceph-node1/keyring
node2> $ sudo -u ceph ceph auth get-or-create mgr.node2 mon 'allow profile mgr' osd 'allow *' mds 'allow *' > /var/lib/ceph/mgr/ceph-node2/keyring

```

启动

```
node0> $ systemctl enable --now ceph-mgr@node0
node0> $ systemctl status ceph-mgr@node0

node1> $ systemctl enable --now ceph-mgr@node1
node1> $ systemctl status ceph-mgr@node1

node2> $ systemctl enable --now ceph-mgr@node2
node2> $ systemctl status ceph-mgr@node2

```

## OSD

将osd密钥发送到osd节点

```
$ scp /var/lib/ceph/bootstrap-osd/ceph.keyring node1:/var/lib/ceph/bootstrap-osd/
$ scp /var/lib/ceph/bootstrap-osd/ceph.keyring node2:/var/lib/ceph/bootstrap-osd/
$ scp /var/lib/ceph/bootstrap-osd/ceph.keyring node3:/var/lib/ceph/bootstrap-osd/
$ scp /var/lib/ceph/bootstrap-osd/ceph.keyring node4:/var/lib/ceph/bootstrap-osd/

```

设置文件权限

```
node1> $ chown -R ceph.ceph /var/lib/ceph/bootstrap-osd
node2> $ chown -R ceph.ceph /var/lib/ceph/bootstrap-osd
node3> $ chown -R ceph.ceph /var/lib/ceph/bootstrap-osd
node4> $ chown -R ceph.ceph /var/lib/ceph/bootstrap-osd

```

在每个osd节点上面执行下面的指令挂载硬盘，记得保存osd id

```
$ ceph-volume lvm create --data /dev/sdb
$ ceph-volume lvm create --data /dev/sdc
$ ceph-volume lvm create --data /dev/sdd
$ ceph-volume lvm create --data /dev/sde

```

启动各个节点osd进程，需要把osd id替换掉，如下在node0上面挂载了四块硬盘，默认从0开始，所以需要启动4个进程

```
$ systemctl restart ceph-osd@0
$ systemctl enable ceph-osd@0

$ systemctl restart ceph-osd@1
$ systemctl enable ceph-osd@1

$ systemctl restart ceph-osd@2
$ systemctl enable ceph-osd@2

$ systemctl restart ceph-osd@3
$ systemctl enable ceph-osd@3

```

可以通过以下指令查询每台主机挂载的osd

```
$ ceph osd tree

```

## mds

只有使用cephfs文件系统时才会需要mds

创建mds数据存储目录

```
node0> sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node0
node1> sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node1
node2> sudo -u ceph mkdir -p /var/lib/ceph/mds/ceph-node2

```

创建mds key

```
node0> $ sudo -u ceph ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node0/keyring --gen-key -n mds.node0
node1> $ sudo -u ceph ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node1/keyring --gen-key -n mds.node1
node2> $ sudo -u ceph ceph-authtool --create-keyring /var/lib/ceph/mds/ceph-node2/keyring --gen-key -n mds.node2

```

最后导入密钥，设置访问权限 同样注意主机名

```
node0> $ sudo -u ceph ceph auth add mds.node0 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node0/keyring
node1> $ sudo -u ceph ceph auth add mds.node1 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node1/keyring
node2> $ sudo -u ceph ceph auth add mds.node2 osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/ceph-node2/keyring

```

node0-2节点修改ceph.conf配置文件，追加以下内容

```
$ cat >> /etc/ceph/ceph.conf <<EOF
[mds.node0]
host = node0

[mds.node1]
host = node1

[mds.node2]
host = node2
EOF

```

node0-2节点重新启动所有服务

```
node0> $ systemctl restart ceph-mon@node0
node0> $ systemctl restart ceph-mgr@node0
node0> $ systemctl restart ceph-mds@node0
node0> $ systemctl enable ceph-mds@node0

node1> $ systemctl restart ceph-mon@node1
node1> $ systemctl restart ceph-mgr@node1
node1> $ systemctl restart ceph-mds@node1
node1> $ systemctl enable ceph-mds@node1

node2> $ systemctl restart ceph-mon@node2
node2> $ systemctl restart ceph-mgr@node2
node2> $ systemctl restart ceph-mds@node2
node2> $ systemctl enable ceph-mds@node2

```

查看集群状态

```
$ ceph -s
  cluster:
    id:     6ad660a2-ddf1-4c85-a852-4a9f789cdfcd
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum node0,node1,node2 (age 20h)
    mgr: node0(active, since 20h), standbys: node1, node2
    mds: 1/1 daemons up, 2 standby
    osd: 20 osds: 20 up (since 20h), 20 in (since 20h)

  data:
    volumes: 1/1 healthy
    pools:   3 pools, 41 pgs
    objects: 43 objects, 6.4 KiB
    usage:   168 MiB used, 100 GiB / 100 GiB avail
    pgs:     41 active+clean

```