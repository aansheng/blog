# CentOS安装配置Zabbix（Nginx+PHP）

Zabbix官方文档中通过[二进制包安装](https://www.zabbix.com/documentation/4.0/zh/manual/installation/install_from_packages/rhel_centos)里面的Zabbix Web是通过Apache+PHP来运行的，但是现在主流的WebServer是Nginx，所以这次我们通过Nginx+PHP的方式来安装。

## 环境

- 系统环境

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ uname -a
Linux ansheng 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

- MySQL

之前我写过一篇文章[CentOS 7安装MySQL 8.0](https://blog.ansheng.me/article/centos-install-mysql-8)，这里我们使用的MySQL是通过`YUM`进行安装的，当然你也可以使用其他安装方式。

## Zabbix Server/Web/Agent

添加Zabbix软件仓库

```bash
rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum-config-manager --enable rhel-7-server-optional-rpms
```

安装

```sql
yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent -y
```

## Zabbix Server

创建zabbix运行所需要的数据库和用户

```bash
$ mysql -uroot -p
Enter password:  # 输入密码

mysql> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected, 2 warnings (0.02 sec)

mysql> CREATE USER 'zabbix'@'localhost' IDENTIFIED BY '2#vTfvc@Y!JQJNJn';
Query OK, 0 rows affected (0.02 sec)

mysql> ALTER USER 'zabbix'@'localhost' IDENTIFIED WITH mysql_native_password BY '2#vTfvc@Y!JQJNJn';
Query OK, 0 rows affected (0.05 sec)

mysql> GRANT ALL privileges ON zabbix.* TO 'zabbix'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

使用MySQL来导入Zabbix Server的初始数据库schema和数据

```bash
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

修改数据库的配置

```bash
$ vi /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=2#vTfvc@Y!JQJNJn
```

运行并设置开机自启动

```bash
systemctl enable --now zabbix-server
```

## Zabbix Agent

```bash
$ vim /etc/zabbix/zabbix_agentd.conf
Server=127.0.0.1
```

启动并设置自启动

```bash
systemctl enable --now zabbix-agent
```

## PHP

安装FastCGI进程管理器（FPM） - php-fpm

安装remi存储库

```bash
rpm -Uhv http://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

激活remi-php71

```bash
yum install -y yum-utils
yum-config-manager --enable remi-php71
```

安装php7.1和需要的模块

```sql
yum install -y php71 php-fpm php-cli php-mysql php-gd php-ldap php-odbc php-pdo php-pecl-memcache php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-bcmath
```

如果安装过程中出现如下错误

```
--> 解决依赖关系完成
错误：软件包：1:php-pear-1.10.9-1.el7.remi.noarch (remi-php71)
          需要：php-composer(fedora/autoloader)
 您可以尝试添加 --skip-broken 选项来解决该问题
 您可以尝试执行：rpm -Va --nofiles --nodigest
```

添加`--skip-broken`参数跳过即可

```bash
yum install -y php71 php-fpm php-cli php-mysql php-gd php-ldap php-odbc php-pdo php-pecl-memcache php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap php-bcmath --skip-broken
```

修改配置文件以通过unix套接字运行它

```bash
$ vim /etc/php-fpm.d/www.conf
# 制定php-fpm运行的用户和组
user = nginx
group = nginx

# 注释这行
;listen = 127.0.0.1:9000

# 添加下面的选项
listen = /var/run/php-fpm/php-fpm.sock
listen.mode = 0660
listen.owner = nginx
listen.group = nginx
```

## Nginx

下载Nginx repo源

```bash
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

```bash
yum install -y nginx
```

修改Nginx默认配置文件

```sql
$ mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf_bak
$ vim /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  _;
    root /usr/share/zabbix;

    location / {
        index index.php index.html index.htm;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "
        max_execution_time = 300
        memory_limit = 128M
        post_max_size = 16M
        upload_max_filesize = 2M
        max_input_time = 300
        date.timezone = Europe/Moscow
        always_populate_raw_post_data = -1
        ";
        fastcgi_buffers 8 256k;
        fastcgi_buffer_size 128k;
        fastcgi_intercept_errors on;
        fastcgi_busy_buffers_size 256k;
        fastcgi_temp_file_write_size 256k;
    }
}
```

上面的配置中如果想更改市区，修改`date.timezone = Europe/Moscow`

检测语法是否正确

```bash
nginx -t
```

启动并设置开机自启动

```sql
systemctl enable --now nginx
```

启动并设置php-fpm机自启动

```sql
systemctl enable --now php-fpm
```

查看运行的sock文件

```bash
$ ll /var/run/php-fpm/php-fpm.sock
srw-rw---- 1 nginx nginx 0 3月  22 13:36 /var/run/php-fpm/php-fpm.sock
```

更改zabbix文件的权限

```bash
chown -R nginx:nginx /var/lib/php/session
chown -R nginx:nginx /etc/zabbix/web
```

然后浏览器打开`http://IP`，进行安装配置吧，`默认的登录账号是Admin，密码zabbix`。