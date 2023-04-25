# PostgreSQL 根据 date/datetime 类型查询的几种方式

下面的实例中`t_create`是`datetime`类型的，需要转换为`date`类型，如果你要查询的字段已经是`date`类型则不需要进行转换.

- 查询当天的数据

通过`cast`函数将`datetime`类型的字段转换为`date`类型，从而进行查找

```sql
select t_create
from orders
where cast(t_create as date) = current_date;
```

- 查询当天数据

如上所示的更简便的方法

```sql
select t_create
from orders
where t_create::date = current_date;
```

- 查询某天

将要查询的日期通过`to_date`转换为`date`类型，然后进行查询

```sql
select t_create
from orders
where t_create::date = to_date('2019-08-08', 'YYYY-MM-DD');
```

- 查询时间范围

通过`between`指定一个日期范围进行查找

```sql
select t_create
from orders
where t_create::date between '2019-08-20' and '2019-08-25';
```

- 查询最近一周的

```sql
select t_create
from orders
where t_create > (now() - interval '1 week');
```
