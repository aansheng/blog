# CentOS Firewalld使用实例

自CentOS 7开始，原先的iptables已经被替换为动态防火墙后台程序firewalld用来做系统层面的防火墙服务，之前的iptables已经被抛弃掉，各大Linux发行版本也相继支持firewalld

## 安装

- 安装firewall

如果是CentOS 7执行以下指令

```
$ yum install firewalld

```

如果是CentOS 8执行以下指令

```
$ dnf install firewalld

```

- 服务相关操作

启动firewall

```
$ systemctl start firewalld

```

设置开机自启

```
$ systemctl enable firewalld

```

删除开机自启

```
$ systemctl disable firewalld

```

停止firewall

```
$ systemctl stop firewalld

```

查看运行状态

```
$ firewall-cmd --state

```

动态更新配置文件

```
$ firewall-cmd --reload

```

## 一些例子

- 查看所有开放的端口

```
$ firewall-cmd --zone=public --list-ports

```

- 开启一个TCP端口
- `-permanent`永久生效，没有此参数重启后失效

```
$ firewall-cmd --zone=public --add-port=80/tcp --permanent

```

- 开启一个范围的端口

```
$ firewall-cmd --zone=public --add-port=9000-9999/tcp --permanent

```

- 删除一个端口

```
$ firewall-cmd --zone=public --remove-port=80/tcp --permanent

```

- 添加一个服务协议

```
$ firewall-cmd --permanent --add-service=http
$ firewall-cmd --permanent --add-service=https

```

- 移除一个服务协议

```
$ firewall-cmd --permanent --remove-service=ssh

```

- 查看开启的服务规则

```
$ firewall-cmd --get-services

```

- 端口转发

开启路由转发

```
$ echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
$ sysctl -p

```

开启防火墙的流量伪装功能

```
$ firewall-cmd --zone=public --permanent --add-masquerade

```

开启TCP流量端口

```
$ firewall-cmd --add-port=8080/tcp --permanent

```

开启UDP流量端口

```
$ firewall-cmd --add-port=8080/udp --permanent

```

开启TCP流量转发

```
$ firewall-cmd --add-forward-port=port=8080:proto=tcp:toaddr=2.2.2.2:toport=666 --permanent

```

开启UDP流量转发

```
$ firewall-cmd --add-forward-port=port=8080:proto=udp:toaddr=2.2.2.2:toport=666 --permanent

```

重载配置文件

```bash
$ firewall-cmd --reload
```