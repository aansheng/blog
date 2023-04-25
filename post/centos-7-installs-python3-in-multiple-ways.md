# CentOS 7多种方式安装Python3

- 环境

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
$ uname -a
Linux ansheng 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

- 安装开发工具包

```bash
$ yum groupinstall -y development tools
```

目前最新的`CentOS 7.6.1810`自带的Python版本只有`Python 2.7.5`

```bash
$ python -V
Python 2.7.5
```

但是目前在工作中，我们都已经使用`Python3`进行开发了，而且每次在项目部署时都要升级到`Python3`，所以还是写篇博客记录下吧。

## EPEL源

[EPEL](https://fedoraproject.org/wiki/EPEL)是`Fedora`小组维护的一个高质量附加软件包。

- 安装epel源

```bash
$ yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

- 搜索Python3版本

```bash
$ yum search python3
```

上面的命令搜索出的最新版本是`Python36`，所以在这里我们会安装`Python36`

- 安装Python36

```bash
$ yum install -y python36 python36-setuptools python36-devel
```

- 安装pip

```bash
$ mkdir -p /usr/local/lib/python3.6/site-packages/  # 需要先创建packages的存放目录，不然安装时会报错
$ easy_install-3.6  pip
```

- 查看版本

```bash
$ python3.6 -V
Python 3.6.6
$ pip3 -V
pip 18.1 from /usr/local/lib/python3.6/site-packages/pip-18.1-py3.6.egg/pip (python 3.6)
```

- 创建虚拟环境

```bash
$ mkdir ~/.venv # 创建虚拟环境目录
$ cd ~/.venv/
$ python3.6 -m venv ansheng  # 创建名为ansheng的虚拟环境
$ source ~/.venv/ansheng/bin/activate  # 切换到ansheng虚拟环境
(ansheng) $ python -V  # python版本
Python 3.6.6
(ansheng) $ pip -V  # pip版本
pip 10.0.1 from /root/.venv/ansheng/lib64/python3.6/site-packages/pip (python 3.6)
(ansheng) $ deactivate  # 退出虚拟环境
```

## 源码

目前EPEL源提供的最新版本也只是`Python3.6`版本，如果要使用目前最新的`Python3.7`版本，那只能从源码编译安装了。

- 安装依赖包

```bash
$ yum install -y gcc openssl-devel bzip2-devel libffi libffi-devel
```

- 下载Python 3.7

你可以从`https://www.python.org/downloads/source/`获取最新的源码包。

```bash
$ cd /usr/src/
$ wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tgz
```

解压

```bash
$ tar xzf Python-3.7.2.tgz
```

- 安装Python 3.7

```bash
$ cd Python-3.7.2
$ ./configure --enable-optimizations
$ make altinstall
```

- 删除下载的源码文件

```bash
$ rm -f /usr/src/Python-3.7.2.tgz
```

- 查看Python与Pip版本

```bash
$ python3.7 -V
Python 3.7.2
$ pip3.7 -V
pip 18.1 from /usr/local/lib/python3.7/site-packages/pip (python 3.7)
```

- 检查yum是否工作

```bash
$ yum install tmux -y
```

到此，安装完成。