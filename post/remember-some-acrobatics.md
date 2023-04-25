# 记一些杂技（持续更新）

- 查看sqlite表结构

```sql
.schema tab_name
```

- Linux制作U盘镜像

```bash
dd if=ISOFILE of=/dev/sdX
```

- Docker中Python中文错误

> UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-8: ordinal not in range(128)

```bash
docker run -it --name python -e PYTHONIOENCODING=utf-8 -e LANG=en_US.utf8 -e LANGUAGE=en_US.utf8 -e LANG_ALL=en_US.utf8 -e LC_ALL=en_US.utf8 centos /bin/bash
```

或者进入容器执行以下指令设置字符集

```bash
export PYTHONIOENCODING=utf-8
export LANG=en_US.utf8
export LANGUAGE=en_US.utf8
export LANG_ALL=en_US.utf8
export LC_ALL=en_US.utf8
```

- 命令行设置代理

```bash
# .bashrc
export http_proxy=http://IP:PORT
export https_proxy=http://IP:PORT
```

- git使用Token

```bash
# .netrc 
machine IP
  login Username
  password Token
```

- pip更新所有包

```bash
pip install -U $(pip freeze | awk '{split($0, a, "=="); print a[1]}')
```

- 普通用户执行docker

```bash
sudo usermod -aG docker your-user
```

- 一键安装Docker

```bash
curl -sSL https://get.docker.com/ | sh  && systemctl enable --now docker
```

- Docker部署MTProxy

```bash
docker pull telegrammessenger/proxy
docker run -d -p 9443:443 --name=mtproto-proxy --ulimit nofile=98304:98304 --restart=always -v proxy-config:/data telegrammessenger/proxy:latest
docker logs mtproto-proxy
```

- 删除celery中所有的任务

命令行

```bash
$ celery -A proj purge
```

或者代码中

```python
from celery.task.control import discard_all
discard_all()
```

`--purge`参数可以再每次启动work时候清理所有等待中的任务

- 取消显示登录

```bash
touch ~/.hushlogin
```

- postgresql create database and user grant

```sql
CREATE DATABASE database_name;
CREATE USER my_username WITH PASSWORD 'my_password';
GRANT ALL PRIVILEGES ON DATABASE "database_name" to my_username;
```
