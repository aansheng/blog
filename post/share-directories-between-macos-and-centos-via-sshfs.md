# 通过SSHFS在MacOS与CentOS之间共享目录（SSH + FTP）

[SSHFS（SSH Filesystem）](https://github.com/osxfuse/sshfs)是一种通过普通ssh连接来挂载和与远程服务器或工作站上的目录和文件交互的文件系统客户端。

然而大多数服务器都默认支持SSH，所以配置起来就非常的简单：服务器什么都不需要操作，在客户端通过SSH挂在服务器的文件目录即可。

# 环境

- 服务器

我这里使用的是`CentOS 7`的系统

```bash
$ whoami
ansheng
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ uname -a
Linux blog 5.0.9-1.el7.elrepo.x86_64 #1 SMP Sat Apr 20 09:03:57 EDT 2019 x86_64 x86_64 x86_64 GNU/Linux
```

我们创建挂载目录以及测试需要用到的数据

```bash
$ mkdir ~/mnt  # 创建挂载目录
$ ll -d ~/mnt  # 注意查看目录权限
drwxrwxr-x 2 ansheng ansheng 4096 4月  28 10:52 /home/ansheng/mnt
$ echo 'Hello, ansheng~' > ~/mnt/test  # 往测试文件写入内容
$ cat ~/mnt/test
Hello, ansheng~
```

- 客户端

我这里是用的macos来做测试的，其他Linux的使用方式也是大致一样的，可以具体参考相关文档。

# MacOS

打开`https://osxfuse.github.io/`这个地址，下载客户端需要的包，然后进行安装，主要是[osxfuse-3.8.3.dmg](https://github.com/osxfuse/osxfuse/releases/download/osxfuse-3.8.3/osxfuse-3.8.3.dmg)和[sshfs-2.5.0.pkg](https://github.com/osxfuse/sshfs/releases/download/osxfuse-sshfs-2.5.0/sshfs-2.5.0.pkg)

- 语法

```bash
$ sshfs username@server:/path on server/ ~/path to mount point.
```

- 查看帮助

```bash
$ sshfs --help
```

- 挂载

首先我们创建要挂载的目录

```bash
$ whoami
shengan
$ pwd
/Users/shengan
$ mkdir ~/mnt
```

下面我将服务器上面`ansheng`家目录下的`mnt`目录挂在到本地`~/mnt`下

```bash
$ sshfs -p9222 ansheng@blog.ansheng.me:/home/ansheng/mnt ~/mnt
ansheng@blog.ansheng.me's password:  # 输入服务器的密码
```

没有报错就表示挂载完成了，然后我们通过`df -h`查看挂载情况

```bash
$ df -h
Filesystem                                  Size   Used  Avail Capacity iused               ifree %iused  Mounted on
ansheng@blog.ansheng.me:/home/ansheng/mnt  9.4Gi  4.2Gi  4.7Gi    48%  146269              482051   23%   /Users/shengan/mnt
```

通过上面的信息我们可以看到，已经挂载成功。

- 测试

首先我们来查看下已经挂载的文件内容

```bash
$ cat ~/mnt/test
Hello, ansheng~
$ ls -lh ~/mnt/test
-rw-rw-r--  1 shengan  staff    16B  4 28 22:54 /Users/shengan/mnt/test
```

文件内容看起来没什么问题，然后我们在客户端上面创建一个文件并写入内容看看服务器会不会有

```bash
$ echo 'Hello, Python~' > ~/mnt/test2
$ cat ~/mnt/test2
Hello, Python~
```

通过服务器进行查看

```bash
$ ls -lh ~/mnt/
总用量 8.0K
-rw-rw-r-- 1 ansheng ansheng 16 4月  28 10:54 test
-rw-r--r-- 1 ansheng ansheng 15 4月  28 11:15 test2
$ cat ~/mnt/test2
Hello, Python~
```

试验成功。

- 卸载

通过`umount`卸载即可

```bash
$ umount ~/mnt
```

通过`df`命令可以看到挂载已经不存在了

```bash
$ ls ~/mnt
$ df -h
Filesystem      Size   Used  Avail Capacity iused               ifree %iused  Mounted on
```

# Linux

`linux`系统只需要安装`sshfs`命令即可，其他的使用方式与`macos`的一样

- On CentOS/RHEL

```bash
$ sudo yum install fuse-sshfs
```

- On Ubuntu & Dabian

```bash
$ sudo apt-get update
$ sudo apt-get install sshfs
```

# 参考文献

1. [osxfuse wiki](https://github.com/osxfuse/osxfuse/wiki/SSHFS)