# CentOS 7安装MySQL 8.0

这`MySQL`的默认端口是`3306`。

Linux上面安装的方式主要分为如下三种：

1. YUM
2. 二进制包
3. 源码包

源码包的安装过于复杂，但是可以灵活配置，这里我们就不介绍了，主要介绍`yum`和`二进制包`的形式。

- 系统环境

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core) 
$ uname -a
Linux host.localdomain 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

# YUM

- 添加YUM源

```bash
$ rpm -ivh https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm
```

mysql repo的下载地址：https://dev.mysql.com/downloads/repo/yum/

- 安装

```bash
$ yum install mysql-community-server -y
```

- 启动并设置开机自启动

```bash
$ systemctl enable --now mysqld
```

- 查看启动状态

```bash
$ systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-12-17 08:47:04 EST; 15min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 11986 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 12055 (mysqld)
   Status: "SERVER_OPERATING"
   CGroup: /system.slice/mysqld.service
           └─12055 /usr/sbin/mysqld

12月 17 08:46:55 host.localdomain systemd[1]: Starting MySQL Server...
12月 17 08:47:04 host.localdomain systemd[1]: Started MySQL Server.
```

- 登录

MySQL在第一次启动的时候，会随机生成一个密码放在`/var/log/mysqld.log`日志文件中，我们可以通过`grep`命令获取到root的密码

```bash
$ grep 'temporary password' /var/log/mysqld.log
2018-06-12T09:34:07.463168Z 1 [Note] A temporary password is generated for root@localhost: WiwB.e2c3udA
```

其中`WiwB.e2c3udA`就是`root`用户的密码，下面我们登录MySQL

```bash
$ mysql -u root -h 127.0.0.1 -p
Enter password:  # 输入密码
mysql>
```

查看数据库

```bash
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

这里会提示我们必须修改root密码才可以正常使用，我们把密码修改为`MyNewPass4!`

```bash
# 修改当前用户的密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
Query OK, 0 rows affected, 1 warning (0.00 sec)
# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.06 sec)
```

然后退出重新登录MySQL

```bash
mysql> exit
Bye
$ mysql -u root -h 127.0.0.1 -p
Enter password:  # 输入刚才设置的密码
# 查看有多少库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
# 查看有多少用户
mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
3 rows in set (0.00 sec)
```

# 二进制包

在安装之前，我们需要安装依赖库，因为MySQL需要

```bash
$ yum install libaio libaio-devel numactl numactl-devel -y
```

最好也把开发工具包安装上

```bash
$ yum groupinstall "Development Tools" -y
```

- 卸载mariadb

```bash
$ yum remove mariadb mariadb-libs -y
```

- 创建MySQL用户和组

```bash
$ groupadd mysql
$ useradd -r -g mysql -s /bin/false mysql
```

- 下载MySQL包

```bash
$ cd /opt/
$ wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.13-linux-glibc2.12-x86_64.tar
```

获取不同版本的下载地址：https://dev.mysql.com/downloads/mysql/


- 解压并创建软链

```bash
$ tar -xvf mysql-8.0.13-linux-glibc2.12-x86_64.tar
$ tar xvf mysql-8.0.13-linux-glibc2.12-x86_64.tar.xz
$ ln -s /opt/mysql-8.0.13-linux-glibc2.12-x86_64 /usr/local/mysql
```

- 创建数据文件存放目录

```bash
$ mkdir /usr/local/mysql/data
```

- 授权

```bash
$ chown -R mysql.mysql /usr/local/mysql
$ ls -ld /usr/local/mysql
lrwxrwxrwx 1 mysql mysql 40 12月 17 09:31 /usr/local/mysql -> /opt/mysql-8.0.13-linux-glibc2.12-x86_64
```

- 添加环境变量

```bash
$ echo 'export PATH=$PATH:/usr/local/mysql/bin' >> ~/.bash_profile
$ source ~/.bash_profile
```

验证是否添加成功

```bash
$ mysql --version
mysql  Ver 8.0.13 for linux-glibc2.12 on x86_64 (MySQL Community Server - GPL)
```

- 初始化数据库

```bash
$ mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
2018-12-17T14:32:55.725037Z 0 [System] [MY-013169] [Server] /opt/mysql-8.0.13-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.13) initializing of server in progress as process 23954
2018-12-17T14:32:59.438919Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: fTVb-*WAq1Oo
2018-12-17T14:33:02.335171Z 0 [System] [MY-013170] [Server] /opt/mysql-8.0.13-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.13) initializing of server has completed
```

记住倒数第二条的信息，告诉你了root的密码是`fTVb-*WAq1Oo`

- 生成SSL

```bash
$ mysql_ssl_rsa_setup --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/
```

- 创建my.cnf参数文件

```bash
$ vim /etc/my.cnf
[mysqld]
port = 3306
user = mysql
socket = /usr/local/mysql/data/mysqld.sock
pid-file = /usr/local/mysql/data/mysqld.pid
basedir = /usr/local/mysql/
datadir = /usr/local/mysql/data/
```

- 设置启动项

```bash
$ vim /usr/lib/systemd/system/mysqld.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

Type=forking

PIDFile=/usr/local/mysql/data/mysqld.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/data/mysqld.pid $MYSQLD_OPTS

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 5000

Restart=on-failure

RestartPreventExitStatus=1

PrivateTmp=false
```


重新加载systemd脚本

```bash
$ systemctl daemon-reload
```

启动并添加开机自启动

```bash
$ systemctl enable --now mysqld
```

查看运行状态

```bash
$ systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2018-12-17 09:35:24 EST; 4s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 24042 ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/data/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
 Main PID: 24044 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─24044 /usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/data/mysqld.pid

12月 17 09:35:22 host.localdomain systemd[1]: Starting MySQL Server...
12月 17 09:35:23 host.localdomain mysqld[24042]: 2018-12-17T14:35:22.445672Z 0 [Warning] [MY-010139] [Server] Changed limits: max_open_files: 5000 (requested 8161)
12月 17 09:35:23 host.localdomain mysqld[24042]: 2018-12-17T14:35:22.445893Z 0 [Warning] [MY-010142] [Server] Changed limits: table_open_cache: 2419 (requested 4000)
12月 17 09:35:23 host.localdomain mysqld[24042]: 2018-12-17T14:35:23.315272Z 0 [System] [MY-010116] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.13) starting as process 24042
12月 17 09:35:24 host.localdomain mysqld[24042]: 2018-12-17T14:35:24.377008Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
12月 17 09:35:24 host.localdomain mysqld[24042]: 2018-12-17T14:35:24.414863Z 0 [System] [MY-010931] [Server] /usr/local/mysql/bin/mysqld: ready for connections. Version: '8.0.13'  socket: '/usr... Server - GPL.
12月 17 09:35:24 host.localdomain systemd[1]: Started MySQL Server.
Hint: Some lines were ellipsized, use -l to show in full.
```

- 连接并修改密码

```bash
$ mysql -uroot -h 127.0.0.1 -p
Enter password:  # 输入密码

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';  # 修改密码为MyNewPass4!
Query OK, 0 rows affected (0.03 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.06 sec)

mysql> exit
Bye
```

连接测试

```bash
$ mysql -uroot -h 127.0.0.1 -p
Enter password:  # 输入刚才设置的密码
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> select user,host from mysql.user;
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
+---------------+-----------+
3 rows in set (0.00 sec)
```

### Docker

如果你的环境比较复杂，搭建起来较为麻烦，没关系，我们可以用docker的方式一键启动，而且还是跨平台的。

- 运行MySQL

```bash
$ docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

上面的指令中，我们运行了`8.x`的`mysql`版本，把`3306`映射出来了，这样可以通过`127.0.0.1:3306`进行访问，其次设置了`root密码为123456`。

> 更多的启动参数可以参考：https://hub.docker.com/_/mysql/