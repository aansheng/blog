# PostgreSQL 备份与恢复

## 备份

备份数据库，需要使用[pg_dump](https://www.postgresql.org/docs/current/backup-dump.html)指令，它将指定的数据库所有内容转储到一个文件中，在数据库服务器上运行入下指令

```bash
$ pg_dump -U db_user -W -F t db_name > /path/to/your/file/dump_name.tar
# -U 指定哪个用户将连接到数据库服务器
# -W 强制pg_dump在连接到服务器之前提示输入密码
# -F 用于指定输出文件的格式，可以是以下之一：
#       p - 纯文本SQL脚本
#       c - 自定义格式存档
#       d - 目录格式存档
#       t - tar格式存档
```

> `p`、`d`和`tar`格式适合使用`pg_restore`进行恢复

## 恢复

数据库的恢复操作有如下两种

### psql

从`pg_dump`创建出来的纯`SQL`文件中进行恢复

```bash
$ psql -U db_user db_name < dump_name.pgsql
```

`db_user`数据库用户，`db_name`是数据库名称，`dump_name.sql`是备份文件的名称

### pg_restore

从`pg_dump`创建出来的`.tar`文件，`目录`或使用的`自定义格式`进行恢复

```bash
$ pg_restore -d db_name /path/to/your/file/dump_name.tar -c -U db_user
# 各种选项，例如：
# -c 在重新创建数据库对象之前删除它们
# -C 在恢复之前创建数据库
# -e 如果遇到错误则退出
# -F format指定存档的格式
```

## 示例

- 备份和还原单个数据库

备份

```bash
$ pg_dump -U postgres -d mydb > mydb.pgsql
```

还原

```bash
$ psql -U postgres -d mydb < mydb.pgsql
```

- 备份和还原所有数据库

备份

```bash
$ pg_dumpall -U postgres > alldbs.pgsql
```

还原

```bash
$ psql -U postgres < alldbs.pgsql
```

- 备份和恢复单表

备份

```bash
$ pg_dump -U postgres -d mydb -t mytable > mydb-mytable.pgsql
```

还原

```bash
$ psql -U postgres -d mydb < mydb-mytable.pgsql
```

- 压缩备份和还原数据库

备份

```bash
$ pg_dump -U postgres -d mydb | gzip > mydb.pgsql.gz
```

从压缩备份文件直接恢复

```bash
$ gunzip -c mydb.pgsql.gz | psql -U postgres -d mydb
```

- 一个简单的备份脚本

```shell
file_name="db-`date +%F-%H-%M-%S`.sql"
export PGPASSWORD="xxxxxxxxxx"

pg_dump -h 127.0.0.1 -U db -w -F p db > ~/.backup/$file_name
```
