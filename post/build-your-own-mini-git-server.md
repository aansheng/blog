搭建自己的小型Git Server

下面我们将会在自己的服务器`（CentOS7）`上面搭建一个自己的`Git服务器`，环境如下：

|IP|主机名|描述|
|:--|:--|:--|
|`192.168.56.150`|server|Git Server|
|`192.168.56.151`|client|Git Client|

- 系统环境

```bash
$ cat /etc/redhat-release 
CentOS Linux release 7.5.1804 (Core) 
$ uname -a                
Linux centos 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
$ whoami 
root
```

在下面的操作中，我们以`hostname`来区分是客户端还是服务端。

## 安装Git

在`server`和`client`两台机器上面都需要安装`git`.

```bash
$ yum install git-core
$ git config --global push.default simple  # 不执行此配置在git push会有警告阻止提交
```

查看`git`版本

```bash
$ git --version
git version 1.8.3.
```

## 创建Git用户

```bash
[root@server ~]# useradd git
[root@server ~]# echo "123456" | passwd --stdin git  # 非交互式方式设置用户密码  
Changing password for user git.
passwd: all authentication tokens updated successfully
```

## 设置授权文件

```bash
[root@server ~]# su - git
Last login: Mon Jun 11 22:44:15 CST 2018 on pts/0
[git@server ~]$ mkdir .ssh
[git@server ~]$ chmod 700 .ssh
[git@server ~]$ touch .ssh/authorized_keys
[git@server ~]$ chmod 600 .ssh/authorized_keys
```

## 创建秘钥

- 生成key

一路回车就好，不需要填其他任何信息

```bash
[root@client ~]# ssh-keygen -C "ianshengme@gmail.com"
```

把key复制到Git Server上面

```bash
[root@client ~]# cat .ssh/id_rsa.pub | ssh git@192.168.56.150 "cat >> ~/.ssh/authorized_keys"                 
The authenticity of host '192.168.56.150 (192.168.56.150)' can't be established.
ECDSA key fingerprint is SHA256:72YotANu9mYYhM2yTj9FUe0WUgmCWJCpzplJCByCTNE.
ECDSA key fingerprint is MD5:99:11:c2:d6:9a:cf:55:4b:5f:fe:9f:e3:b4:6d:00:cd.
Are you sure you want to continue connecting (yes/no)? yes  # 第一次连接会有此提示,yes即可
Warning: Permanently added '192.168.56.150' (ECDSA) to the list of known hosts.
git@192.168.56.150's password:  # 输入刚才设置的密码123456
```

- 测试是否可以连接

```bash
# 登录成功表示秘钥配置的没问题
[root@client ~]# ssh git@192.168.56.150
Last login: Mon Jun 11 22:46:52 2018
[git@server ~]$ logout  # 退出
Connection to 192.168.56.150 closed
```

## 创建仓库

```bash
[git@server ~]$ git init --bare blog.git      
Initialized empty Git repository in /home/git/blog.git/
```

## 克隆仓库

- clone仓库

```bash
[root@client ~]# git clone git@192.168.56.150:/home/git/blog.git      
Cloning into 'blog'...
warning: You appear to have cloned an empty repository.
[root@client ~]# ls -l
total 0
drwxr-xr-x 3 root root 35 Jun 11 22:56 blog
[root@client ~]# cd blog/
[root@client blog]# ll -a
total 0
drwxr-xr-x  3 root root  18 Jun 11 22:53 .
dr-xr-x---. 5 root root 164 Jun 11 22:53 ..
drwxr-xr-x  7 root root 119 Jun 11 22:53 .git
```

- 配置git客户端信息

```bash
[root@client blog]# git config user.name "ansheng"
[root@client blog]# git config user.email "ianshengme@gmail.com
```

- 添加测试文件

```bash
[root@client blog]# echo "Hello, ansheng" > README.md 
[root@client blog]# ls
README.md
```

- 提交

```bash
[root@client blog]# git add .
[root@client blog]# git commit -m "Create README.md"
[root@client blog]# git push
Counting objects: 3, done.
Writing objects: 100% (3/3), 231 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@192.168.56.150:/home/git/blog.git
 * [new branch]      master -> master
```
