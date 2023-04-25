# CentOS Linux部署NFS服务

## 环境

|主机名|IP|描述|
|:--|:--|:--|
|server|192.168.2.20|NFS服务端|
|client|192.168.2.21|NFS客户端|

- 系统版本

```
$ cat /etc/redhat-release
CentOS Linux release 8.3.2011
$ uname -a
Linux server 4.18.0-240.1.1.el8_3.x86_64 #1 SMP Thu Nov 19 17:20:08 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

```

- 关闭Firewalled

```
sudo systemctl disable --now firewalld

```

- 关闭SELinux

```
sudo sed -i s/^SELINUX=.*$/SELINUX=disabled/ /etc/selinux/config

```

- 重启

```
sudo shutdown -r now

```

## 安装NFS软件包

- 安装

```
sudo dnf install nfs-utils -y

```

- 启动

```
sudo systemctl enable --now nfs-server

```

- 查看支持的nfs协议版本

```
$ cat /proc/fs/nfsd/versions
-2 +3 +4 +4.1 +4.2

```

## 服务端配置

- 创建共享目录

```
sudo mkdir /mnt/nfs_share -p

```

- 配置权限

```
$ sudo chown nobody.nobody /mnt/nfs_share
$ ls -ld /mnt/nfs_share
drwxr-xr-x 2 nobody nobody 6 Aug 11 04:21 /mnt/nfs_share

```

- 修改配置文件

```
$ sudo vi /etc/exports
/mnt/nfs_share    192.168.2.21(rw,sync,no_all_squash,root_squash)

```

- 重启nfs服务

```
sudo systemctl restart nfs-server

```

## 客户端配置

- 查看服务端共享的目录

```
$ showmount -e 192.168.2.20
Export list for 192.168.2.20:
/mnt/nfs_share 192.168.2.21

```

- 挂载到本地

```
$ sudo mount -t nfs 192.168.2.20:/mnt/nfs_share /mnt
$ df -h | grep /mnt/
192.168.2.20:/mnt/nfs_share   10G  3.4G  6.7G  34% /mnt

```

- 写入文件测试

```
$ sudo touch /mnt/test.txt
$ ls -l /mnt/test.txt
-rw-r--r-- 1 nobody nobody 0 Aug 11 04:34 /mnt/test.txt

```

- 写入fstab开机自启动

```
$ sudo vi /etc/fstab
192.168.2.20:/mnt/nfs_share /mnt       nfs   defaults,timeo=900	0 0

```

- 重启查看

```
$ sudo shutdown -r now
$ df -h | grep /mnt/
192.168.2.20:/mnt/nfs_share   10G  3.4G  6.7G  34% /mnt

```

- 卸载

```
$ sudo umount /mnt/
$ df -h | grep /mnt/

```