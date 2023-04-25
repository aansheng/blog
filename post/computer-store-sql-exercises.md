# SQL语句练习-电脑商店

# 创建表

- 商店表

```sql
create table manufacturers
(
    id   integer primary key,  -- 商店ID
    name varchar(100) not null -- 商店名称
);
```

- 商品表

```sql
create table products
(
    id            integer primary key,                  --商品ID
    name          varchar(100) not null,                --商品名称
    price         decimal      not null,                --商品价格
    manufacturer integer references manufacturers (id)  --关联商店
);
```

# 预置数据

- 商店信息

```sql
insert into manufacturers
values (1, '索尼'),
       (2, '华硕'),
       (3, '惠普'),
       (4, '英特尔'),
       (5, '富士通'),
       (6, '金士顿');
```

- 商品信息

```sql
insert into products
values (1, '硬盘', 240, 5),
       (2, '内存', 120, 6),
       (3, 'CPU', 150, 4),
       (4, 'U盘', 5, 6),
       (5, '监控', 240, 1),
       (6, 'DVD驱动器', 180, 2),
       (7, 'CD驱动器', 90, 2),
       (8, '打印机', 270, 3),
       (9, '墨粉盒', 66, 3),
       (10, 'DVD刻录机', 180, 2);
```

# 题目

1. 选择商店中所有产品的名称
2. 选择商店中所有产品的名称和价格
3. 选择价格小于或等于200的产品名称
4. 选择价格在60到120之间的所有产品
5. 选择名称和价格，但是价格必须乘以100
6. 计算所有产品的平均价格
7. 计算制造商ID等于2的所有产品的平均价格
8. 计算价格大于或等于180的产品数量
9. 选择价格大于或等于180的所有产品的名称和价格，然后按价格降序排序，然后按名称升序排序
10. 选择产品中的所有数据，包括每个产品制造商的所有数据
11. 选择所有产品的产品名称，价格和制造商名称
12. 选择每个制造商产品的平均价格，仅显示制造商的代码
13. 选择每个制造商产品的平均价格，显示制造商的名称
14. 选择产品的平均价格大于或等于150美元的制造商名称
15. 选择最便宜产品的名称和价格
16. 选择每个制造商的名称以及最昂贵产品的名称和价格
17. 添加新产品：音箱, 价格：70, 制造商ID：2
18. 将产品id为8的名称更新为`激光打印机`
19. 对所有产品享受10％的折扣
20. 对价格大于或等于120美元的所有产品申请10％的折扣

# 答案

1. 选择商店中所有产品的名称

```sql
select name
from products;
```

2. 选择商店中所有产品的名称和价格

```sql
select name, price
from products;
```

3. 选择价格小于或等于200的产品名称

```sql
select name
from products
where price <= 200;
```

4. 选择价格在60到120之间的所有产品

```sql
/* With AND */
select *
from products
where price >= 60
  and price <= 120;

/* With BETWEEN */
select *
from products
where price between 60 and 120;
```

5. 选择名称和价格，但是价格必须乘以100

```sql
select name, price * 100 as price
from products;
```

6. 计算所有产品的平均价格

```sql
select avg(price)
from products;
```

7. 计算制造商ID等于2的所有产品的平均价格

```sql
select avg(price)
from products
where manufacturer = 2;
```

8. 计算价格大于或等于180的产品数量

```sql
select count(*)
from products
where price >= 180;
```

9. 选择价格大于或等于180的所有产品的名称和价格，然后按价格降序排序，然后按名称升序排序

```sql
select name, price
from products
where price >= 180
order by price desc, name;
```

10. 选择产品中的所有数据，包括每个产品制造商的所有数据

```sql
/* Without INNER JOIN */
select *
from products as p,
     manufacturers as m
where p.manufacturer = m.id;

/* With INNER JOIN */
select *
from products as p
         inner join manufacturers m on p.manufacturer = m.id;
```

11. 选择所有产品的产品名称，价格和制造商名称

```sql
/* Without INNER JOIN */
select p.name, p.price, m.name
from products as p,
     manufacturers as m
where p.manufacturer = m.id;

/* With INNER JOIN */
SELECT p.Name, p.Price, m.Name
FROM products as p
         INNER JOIN manufacturers as m
                    ON p.Manufacturer = m.id;
```

12. 选择每个制造商产品的平均价格，仅显示制造商的代码

```sql
select avg(price), manufacturer
from products
group by manufacturer;
```

13. 选择每个制造商产品的平均价格，显示制造商的名称

```sql
/* Without INNER JOIN */
select avg(price), m.name
from products as p,
     manufacturers as m
where p.manufacturer = m.id
group by m.name

/* With INNER JOIN */
select avg(price), m.name
from products as p
         inner join manufacturers m on p.manufacturer = m.id
group by m.name
```

14. 选择产品的平均价格大于或等于150美元的制造商名称

```sql
/* Without INNER JOIN */
select avg(Price), m.name
from products as p,
     manufacturers as m
where p.manufacturer = m.id
group by m.name
having avg(Price) >= 150;

/* With INNER JOIN */
SELECT AVG(Price), m.Name
FROM Products as p
         INNER JOIN manufacturers as m
                    ON p.Manufacturer = m.id
GROUP BY m.Name
HAVING AVG(Price) >= 150;
```

15. 选择最便宜产品的名称和价格

```sql
SELECT name, price
FROM Products
ORDER BY price ASC
LIMIT 1;

/* With a nested SELECT */
SELECT Name, Price
FROM Products
WHERE Price = (SELECT MIN(Price) FROM Products);
```

16. 选择每个制造商的名称以及最昂贵产品的名称和价格

```sql
/* With a nested SELECT and without INNER JOIN */
SELECT A.Name, A.Price, F.Name
FROM Products A,
     Manufacturers F
WHERE A.Manufacturer = F.id
  AND A.Price =
      (
          SELECT MAX(A.Price)
          FROM Products A
          WHERE A.manufacturer = F.id
      );

/* With a nested SELECT and an INNER JOIN */
SELECT A.Name, A.Price, F.Name
FROM Products A
         INNER JOIN manufacturers F
                    ON A.Manufacturer = F.id
                        AND A.Price =
                            (
                                SELECT MAX(A.Price)
                                FROM Products A
                                WHERE A.Manufacturer = F.id
                            );
```

17. 添加新产品：音箱, 价格：70, 制造商ID：2

```sql
insert into products
values (11, '音箱', 70, 2);
```

18. 将产品id为8的名称更新为`激光打印机`

```sql
update products
set name='激光打印机'
where id = 8;
```

19. 对所有产品享受10％的折扣

```sql
UPDATE Products
SET Price = Price - (Price * 0.1);
```

20. 对价格大于或等于120美元的所有产品申请10％的折扣

```sql
update products
set price=price - (price * 0.1)
where price >= 120;
```