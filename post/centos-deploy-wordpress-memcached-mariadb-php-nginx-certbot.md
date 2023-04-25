# CentOS部署Wordpress(Memcached+MariaDB+PHP+Nginx+CertBOT)

转了一大圈博客最终还是放在[Wordpress](https://cn.wordpress.org/)上面了，感觉不用操心把，乱七八糟的东西也懒得折腾，嗯...好像是这样的，在此记录一下博客搭建的过程，以便后续维护、迁移的时候也可以回顾回顾。

## 环境

下面的环境均在CentOS 8上面操作的

```
$ whoami
root
$ cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)

```

然后系统尽量维持在最新版

```
$ dnf makecache
$ dnf update -y

```

建议关闭SELinux和Firewalld，防火墙可以使用云厂商自带的

```
$ vim /etc/selinux/config
SELINUX=disabled
$ systemctl disable --now firewalld

```

最后重启保证修改生效

```
$ reboot

```

## MariaDB

MariaDB和MySQL一样，但是MySQL新版占用内存太高了，我的1G内存小鸡可能扛不太住，所以就算则MariaDB了，相对来说更轻量，资源占用率比较低。

- 安装MariaDB

```
$ dnf install mariadb-server -y

```

- 开机启动mariadb

```
$ systemctl enable --now mariadb

```

- 初始化

```
$ mysql_secure_installation
......
Enter current password for root (enter for none): # 直接回车即可，因为我没还没有设置root密码
......
Set root password? [Y/n] Y # 设置ROOT密码
New password: # 输入root密码，我这里输入的是：#1nKf4D^0NGPb*Ak
Re-enter new password:
Password updated successfully!
......
Remove anonymous users? [Y/n] Y # 移除匿名用户
......
Disallow root login remotely? [Y/n] Y # 关闭root远程登录
......
Remove test database and access to it? [Y/n] Y # 移除测试数据库
......
Reload privilege tables now? [Y/n] Y # 刷新权限表

```

- 设置字符编码为utf8

编辑配置文件我们需要修改字符集为utf8

```
$ vim /etc/my.cnf.d/mariadb-server.cnf
......
[mysqld]
......
# 在mysqld段增加下面的配置
character-set-server=utf8
collation-server=utf8_unicode_ci
......

```

重启服务

```
$ systemctl restart mariadb

```

- 创建Wordpress数据库

```
$ mysql -uroot -p
# 输入root密码
Enter password:
# 查看字符编码
MariaDB [(none)]> \\s
--------------
mysql  Ver 15.1 Distrib 10.3.17-MariaDB, for Linux (x86_64) using readline 5.1
Connection id:          8
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server:                 MariaDB
Server version:         10.3.17-MariaDB MariaDB Server
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8
Db     characterset:    utf8
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /var/lib/mysql/mysql.sock
Uptime:                 1 min 10 sec
Threads: 7  Questions: 4  Slow queries: 0  Opens: 17  Flush tables: 1  Open tables: 11  Queries per second avg: 0.057
--------------
# 创建wp数据库
MariaDB [(none)]> create database wp;
Query OK, 1 row affected (0.000 sec)

```

## Memcached

为了加速我们的网站访问，减少查询，我们用了Memcached来做缓存服务，至于为什么不用redis，因为Memcached足以满足我们的服务

- 安装memcached

```
$ dnf install memcached libmemcached -y

```

- 修改配置文件使只监听127.0.0.1

```
$ vim /etc/sysconfig/memcached
......
OPTIONS="-l 127.0.0.1"
......

```

- 开机启动mariadb

```
$ systemctl enable memcached --now

```

## PHP

PHP这里用的`7.4`版本，最好还是不要使用老版本吧

- 安装

```
$ dnf install epel-release -y
$ dnf install <https://rpms.remirepo.net/enterprise/remi-release-8.rpm> -y
$ dnf module reset php
$ dnf module enable php:remi-7.4 -y
$ dnf install php-pecl-memcached php-pecl-memcache php php-opcache php-gd php-curl php-mysqlnd php-zip php-mbstring php-devel php-json -y

```

这里先把nginx服务安装了

```
$ dnf install nginx -y

```

- 修改配置

默认使用的用户是apache，但是我们使用的是nginx，所以需要把用户修改为nginx

```
$ vim /etc/php-fpm.d/www.conf
......
user = nginx
group = nginx
......

```

- 修改文件授权

```
$ chown -R root:nginx /var/lib/php

```

- 启动

```
$ systemctl enable --now php-fpm

```

## Wordpress

Wordpress下载解压到指定目录下就可以了

```
# 安装wgey命令，下面需要用到
$ dnf install wget -y
$ cd /var/www
$ wget <https://wordpress.org/latest.tar.gz>
$ rm -fr html
$ tar xf latest.tar.gz
$ mv wordpress html
$ rm -f latest.tar.gz

```

- 设置授权

```
$ chown -R nginx.nginx html

```

- 数据库配置

```
$ cd html/
$ cp wp-config-sample.php wp-config.php
$ chown nginx.nginx wp-config.php
$ vim wp-config.php
......
define( 'DB_NAME', 'wp' );
define( 'DB_USER', 'root' );
define( 'DB_PASSWORD', '#1nKf4D^0NGPb*Ak' );
define( 'DB_HOST', 'localhost' );
......

```

- SEO和Google统计（可选）

如果你的主题不提供SEO的配置，你可以编辑主题目录下面的header.php文件中的head标签中添加一下代码

```
vim wp-content/themes/neve/header.php
<!-- keywords and description -->
<?php if (is_home()) {
    $description = "安生(anSheng.Me)，记录工作中可能遇到的django、djangorestframework、postgresql、docker问题，分享python教程、django教程";
    $keywords = "安生,ansheng,python,python教程,django,django教程,djangorestframework,centos,celery,postgresql,docker";
} elseif (is_single()) {
    if ($post->post_excerpt) {
        $description     = $post->post_excerpt;
    } else {
        $description = substr(strip_tags($post->post_content),0,220);
    }
    $keywords = "";
    $tags = wp_get_post_tags($post->ID);
    foreach ($tags as $tag ) {
        $keywords = $keywords . $tag->name . ",";
    }
    if (substr($keywords, -1) == ",") {
      $keywords = substr($keywords, 0, -1);
    }
} elseif (is_category()) {
    $description = category_description();
    $keywords = single_cat_title('', false);
} elseif (is_tag()) {
    $keywords = single_tag_title('', false);
    $description = "关于标签 " . $keywords . " 的相关文章";
}
$keywords = trim(strip_tags($keywords));
$description = trim(strip_tags($description));
?>
<meta name="keywords" content="<?=$keywords?>" />
<meta name="description" content="<?=$description?>" />
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="<https://www.googletagmanager.com/gtag/js?id={GOOGLE_UA_ID}>"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '{GOOGLE_UA_ID}');
</script>

```

## Nginx

nginx在PHP部分已经安装好了，所以直接跳过安装进入部署

- 部署

```
$ vim /etc/nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;

    server {
        listen 80;
        server_name ansheng.me;

        root /var/www/html;
        index index.php;

        location / {
           try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \\.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\\.php)(/.+)$;
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location = /xmlrpc.php {
            deny all;
            access_log off;
        }

        location = /favicon.ico {
            log_not_found off;
            access_log off;
        }

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }

        location ~ /\\. {
            deny all;
        }

        location ~* /(?:uploads|files)/.*\\.php$ {
            deny all;
        }

        location ~* \\.(js|css|png|jpg|jpeg|gif|ico)$ {
            expires max;
            log_not_found off;
        }
    }
}

```

- 启动NGINX

```
$ systemctl enable --now  nginx

```

- 添加域名解析

我这里使用的域名是`ansheng.me`，需要在域名服务商后台添加一条DNS A记录指向到服务地址，我这里已经添加好了。

- 初始化wordpress

浏览器打开`ansheng.me`，一步一步初始化

选择语言

![Untitled](/images/2020/10/18/1.png)

设置站点信息

![Untitled](/images/2020/10/18/2.png)

初始化成功之后点击登陆，至此WP安装已经完成了

![Untitled](/images/2020/10/18/3.png)

Wordpress配置Memcached你可以安装[W3 Total Cache](https://cn.wordpress.org/plugins/w3-total-cache/)插件，具体配置比较简单，这里就不说了

## CertBOT

如果你的站点并不想使用https，那么可以直接跳过这段。

为了使用https我们需要通过使用certbot来签发ssl整数

```
$ curl -O <https://dl.eff.org/certbot-auto>
$ mv certbot-auto /usr/local/bin/certbot-auto
$ chmod 0755 /usr/local/bin/certbot-auto
# 根据提示输入一些信息即可
$ /usr/local/bin/certbot-auto --nginx

```

由于签发的证书有效期只有三个月，所以我们需要添加定时任务来自动续签

```
$ crontab -e
0 0,12 * * * /usr/local/bin/certbot-auto renew --dry-run

```

下面是使用了https的nginx完整的配置文件

```
$ vim /etc/nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    # 添加gzip压缩
    gzip on;
    gzip_min_length 0;
    gzip_comp_level 9;
    gzip_types *;
    gzip_vary on;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;

    # 隐藏nginx版本信息
    server_tokens off;

    server {
        if ($host = ansheng.me) {
            return 301 https://$host$request_uri;
        }

        listen 80;
        server_name ansheng.me;
        return 404;
    }

    server {
        listen 443 ssl http2;
        server_name ansheng.me;

        ssl_certificate /etc/letsencrypt/live/ansheng.me/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ansheng.me/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
        ssl_session_cache shared:SSL:20m;

        root /var/www/html;
        index index.php;

        location / {
           try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \\.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\\.php)(/.+)$;
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location = /xmlrpc.php {
            deny all;
            access_log off;
        }

        location = /favicon.ico {
            log_not_found off;
            access_log off;
        }

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }

        location ~ /\\. {
            deny all;
        }

        location ~* /(?:uploads|files)/.*\\.php$ {
            deny all;
        }

        location ~* \\.(js|css|png|jpg|jpeg|gif|ico)$ {
            expires max;
            log_not_found off;
        }
    }
}

```

当访问http会自动跳转到https

```
$ curl -I ansheng.me
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sun, 18 Oct 2020 04:09:15 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: <https://ansheng.me/>

```

https可以正常访问

```
$ curl -I <https://ansheng.me/>
HTTP/2 200
server: nginx
date: Sun, 18 Oct 2020 04:09:23 GMT
content-type: text/html; charset=UTF-8
vary: Accept-Encoding
x-powered-by: PHP/7.4.11
link: <https://ansheng.me/wp-json/>; rel="<https://api.w.org/>"
strict-transport-security: max-age=31536000; includeSubDomains

```

当Memcached缓存配置之后，访问几次页面，然后通过telnet连接memcached查看是否生效