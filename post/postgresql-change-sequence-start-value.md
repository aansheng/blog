# PostGresql更改自增ID值

不知道你会不会遇到类似这样的情况，比如有一个`orders(订单)`表，我们把ID作为订单的ID，方便查询以及前端的显示，但是默认情况下，ID自增是从1开始的，为了前端显示的效果通常我们会将ID从6(N)位数开始，这个时候就需要修改默认的自增起始值了。

基于这种情况我们来做以下的演示操作。

# 环境

- 运行postgresql

我这里通过docker的方式启动postgresql

```bash
$ docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=ansheng.me postgres
```

默认的密码为`ansheng.me`

- 版本

连接到postgres之后就会显示版本

```bash
$ psql -h localhost -U postgres
Password for user postgres:  # 输入密码
psql (11.2)
Type "help" for help.

postgres=# exit
```

# 创建建orders表

- 创建orders表

```sql
create table orders
(
    id           serial,
    product_name varchar(100)
);
```

通过`serial`创建自增ID，同时会创建属于这个表的自增序列，名为:`orders_id_seq`，并将ID字段设置为`not null`

- 查看序列名称

使用命令：`\d 表名`

```sql
postgres=# \d orders
                                       Table "public.orders"
    Column    |          Type          | Collation | Nullable |              Default
--------------+------------------------+-----------+----------+------------------------------------
 id           | integer                |           | not null | nextval('orders_id_seq'::regclass)
 product_name | character varying(100) |           |          |
```

其中`orders_id_seq`就是序列的名称

- 添加测试数据

```sql
insert into orders(product_name)
values ('MacMini'),
       ('MacBook'),
       ('iPhone');
```

- 查看数据

```sql
postgres=# select * from orders;
 id | product_name
----+--------------
  1 | MacMini
  2 | MacBook
  3 | iPhone
(3 rows)
```

上述的数据ID是从1开始，然后依次递增。

# 更改自增ID的起始值

```sql
postgres=# SELECT setval('orders_id_seq', 100000, true); -- 下一个ID值为100001
 setval
--------
 100000
(1 row)
```

如果`setval`的第三个值不添加为true，下一个ID值为`100000`，设置为true则表示设置的当前序列值+1

- 插入数据

```sql
postgres=# insert into orders(product_name) values ('iPad');
INSERT 0 1
```

- 查看

```sql
postgres=# select * from orders;
   id   | product_name
--------+--------------
      1 | MacMini
      2 | MacBook
      3 | iPhone
 100001 | iPad
(4 rows)
```

可以看到`iPad`的ID为`100001`

# 把当前最大的ID做为当前的ID自增起始值

```sql
postgres=# select setval('orders_id_seq', (select max(id) from orders));
 setval
--------
 100001
(1 row)
```

# 参考文献

- [Sequence Manipulation Functions](https://www.postgresql.org/docs/11/functions-sequence.html)
- [Postgres manually alter sequence](https://stackoverflow.com/questions/8745051/postgres-manually-alter-sequence)