# PostgreSQL 基础教程

## 插入数据

假设`users`表只有`first_name`,`last_name`和`email`这三个列

```sql
insert into users
values ('John', 'Doe', 'john@doe.com');
```

你也可以指定需要插入的列，但前提是其他列可以为空

```sql
insert into users (first_name)
values ('John');
```

为列插入`json`数据

```sql
insert into users (preferences)
values ('{ "beta": true }');
```

如果插入行会违反唯一约束，则可以使用`Postgres'on conflict`子句指定发生这种情况时要执行的操作

```sql
-- 如果已经记录了这个webhook，那么什么都不做
insert into stripe_webhooks (event_id)
values ('evt_123')
on conflict do nothing;
```

您还可以在 Postgres 中执行`upserts（更新或插入）`

```sql
-- 假设email列是唯一索引
insert into users (email, name)
values ('john@doe.com', 'Jane Doe')
on conflict (email) do update set name = excluded.name; -- excluded.name指的是'Jane Doe'
```

## 更新数据

```sql
-- 更新所有行
update users set updated_at = now();

-- 更新指定行
update users set updated_at = now() where id = 1;
```

## 删除数据

```sql
delete from users where id = 1;
```

## 创建表

以下是创建`users`表的示例：

```sql
create table users
(
    id          serial primary key,         -- 自增长ID
    name        character varying,          -- 字符串列，为指定长度
    preferences jsonb,                      -- JSON列
    created_at  timestamp without time zone -- 始终以utc格式存储
);
```

为列指定`非空约束`和`默认值`

```sql
create table users
(
    id     serial primary key,
    name   character varying not null,
    active boolean default true
);
```

创建临时表，这些表将在会话期间保持不变，会话结束之后将会被删除

```sql
-- 创建一个名为`scratch_users`的临时表，只有一个`id`列
create temporary table scratch_users
(
    id integer
);

-- 或者根据select的输出创建临时表
create temp table active_users
as
select *
from users
where active is true;
```

## 删除表

```sql
drop table funky_users;
```

## 重命名表

```sql
alter table events rename to events_backup;
```

## 清空表

```sql
truncate my_table
```

如果你有一个`ID`自增列，并且想重新启动它的序列（即重新启动 ID 1）

```sql
truncate my_table restart identity
```

## 复制表

```sql
create table dupe_users as (select *
                            from users);
```

只创建表结构不添加数据

```sql
create table dupe_users as (select *
                            from users) with no data;
```

## 添加列

向 users 表中添加 created_at 时间戳列的示例

```sql
alter table users
    add column created_at timestamp without time zone;
```

添加非空约束的字符串（varchar）列，表中无数据时可执行

```sql
alter table users
    add column bio character varying not null;
```

添加具有默认值的布尔列

```sql
alter table users
    add column active boolean default true;
```

## 删除列

```sql
alter table users
    drop column created_at;
```

## 重命名列

```sql
alter table users
    rename column registered_at to created_at;
```

## 为列添加默认值

```sql
-- Example: 订单的默认总计为0
alter table orders
    alter column total_cents set default 0;

-- Example: 默认Items可用
alter table items
    alter column available set default true;
```

## 列中删除默认值

假设`orders.total_cents`有一个默认值，这将删除未来插入的默认值

```sql
alter table orders
    alter column total_cents drop default;
```

## 为列添加一个非空约束

```sql
alter table users
    alter column email set not null;
```

## 删除列中的非空约束

```sql
alter table users
    alter column email drop not null;
```

## 创建索引

在拥有大量数据时，创建正确的索引对于高性能查询至关重要。

```sql
create index concurrently 'index_created_at_on_users' on users using btree (created_at);
```

为多列创建索引

```sql
create index concurrently 'index_user_id_and_time_on_events' on events using btree (user_id, time);
```

防止数据重复的唯一索引

```sql
create unique index concurrently 'index_stripe_event_id_on_stripe_events' on stripe_events using btree (stripe_event_id);
```

创建满足特定条件行的索引

```sql
create index concurrently 'index_active_users' on users using btree (created_at) where active is true;
```

还可以拥有唯一的部分索引，例如，假设每个用户只能拥有一张有效的信用卡

```sql
-- 这将阻止用户拥有多个激活的银行卡
create unique index concurrently 'index_active_credit_cards' on credit_cards using btree(user_id) where active is true;
```

## 删除索引

```sql
drop index index_created_at_on_users;
```

## 创建视图

```sql
create or replace view enriched_users as (
    select *
    from users
             inner join enrichments on enrichments.user_id = users.id
);
```

## 删除视图

```sql
drop view enriched_users;
```

## 按照时间进行分组

如果你想按分、时、日、周等进行分组，需要使用`PostgreSQL`函数`date_trunc`

```sql
select date_trunc('minute', created_at), -- or hour, day, week, month, year
       count(1)
from users
group by 1
```

## 截断时间戳

```sql
select date_trunc('second', now()) -- or minute, hour, day, month
```

## 将 UTC 转换为本地时区

如果您有一个没有时区列的时间戳，并且您将时间戳存储为`UTC`，则需要告诉 PostgreSQL`，然后告诉它将其转换为您的本地时区。

```sql
select created_at at time zone 'utc' at time zone 'america/los_angeles'
from users;
```

为了更简洁，您还可以使用时区的缩写：

```sql
select created_at at time zone 'utc' at time zone 'pst'
from users;
```

要查看 PostgreSQL 支持的时区列表：

```sql
select *
from pg_timezone_names;
```

## 类型转换

```sql
-- 文本转换诶布尔值
select 'true'::boolean;

-- 浮点数转换为整型
select 1.0::integer;

-- 整型转换为浮点数
select '3.33'::float;
select 10 / 3.0;
-- 整型和浮点数进行计算将返回浮点数

-- 文本转换为整型
select '1'::integer;

-- 文本转换诶时间戳
select '2018-01-01 09:00:00'::timestamp;

-- 文本转换为日期
select '2018-01-01'::date;

-- Cast text to interval
select '1 minute'::interval;
select '1 hour'::interval;
select '1 day'::interval;
select '1 week'::interval;
select '1 month'::interval;
```

## 从 CSV 中导入数据

```sql
-- Assuming you have already created an imported_users table
-- Assuming your CSV has no headers
\copy imported_users from 'imported_users.csv' csv;

-- If your CSV does have headers, they need to match the columns in your table
\copy imported_users from 'imported_users.csv' csv header;

-- If you want to only import certain columns
\copy imported_users (id, email) from 'imported_users.csv' csv header;
```

## Coalesce

```sql
select
  day,
  tickets
from stats;
    day     | tickets
------------+-------
 2018-01-01 |     1
 2018-01-02 |   null
 2018-01-03 |     3
```

使用`coalesce`函数，该函数返回它传递的第一个非`null`参数

```sql
select
  day,
  coalesce(tickets, 0)
from stats;
    day     | tickets
------------+-------
 2018-01-01 |     1
 2018-01-02 |     0
 2018-01-03 |     3
```

## Case 语句

```sql
select
  case
    when precipitation = 0 then 'none'
    when precipitation <= 5 then 'little'
    when precipitation > 5 then 'lots'
    else 'unknown'
  end as amount_of_rain
from weather_data;
```

## 列使用 filter 进行计数

```sql
select
  count(1), -- Count all users
  count(1) filter (where gender = 'male'), -- Count male users
  count(1) filter (where beta is true) -- Count beta users
  count(1) filter (where active is true and beta is false) -- Count active non-beta users
from users
```

## 创建用户及授权

```sql
create database DB_NAME;
create user DB_USER with encrypted password 'USER_PASSWORD';
grant all privileges on database DB_NAME to DB_USER;
```

```sql
\du
alter role <user-name> superuser;
```
