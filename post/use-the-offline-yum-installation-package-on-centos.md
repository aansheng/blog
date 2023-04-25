# 在CentOS上使用离线YUM安装软件包

## 下载软件包（有网络环境）

创建下载的软件包存放目录

```bash
$ mkdir -p /root/download
```

获取`createrepo`和`tmux`安装包

```bash
$ yum install --downloadonly --downloaddir=/root/download createrepo tmux
```

安装`createrepo`

```bash
$ yum install createrepo -y
```

初始化`yum`源的`repodata`

```bash
$ createrepo -pdo /root/download /root/download
$ createrepo --update /root/download
```

打包`download`目录并把打包之后的文件放到没有网络的服务器上

```bash
$ tar zcf download.tar.gz download/
```

## 安装软件包（无网络环境）

解压软件包

```bash
$ tar xf download.tar.gz
```

移除默认的repo配置

```bash
$ cd /etc/yum.repos.d
$ mkdir bak
$ mv CentOS-* bak
```

进行yum客户端配置

```bash
$ vi /etc/yum.repos.d/localyum.repo
[localyum]
name=localyum
baseurl=file:///root/download
enable=1
gpgcheck=0
```

更新缓存

```bash
$ yum clean all
$ yum makecache
```

- 安装createrepo

```bash
$ yum install createrepo
```

安装tmux

```bash
$ yum install tmux
```

如果`/root/download`下有新增的`rpm`安装包，请使用以下命令更新`repo`

```bash
$ createrepo --update /root/download
```

## 参考

- [CentOS7 docker-engine 完全离线安装](https://my.oschina.net/u/3133713/blog/808441)
