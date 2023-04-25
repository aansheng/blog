# SQL语句练习-仓库

# 创建表

- 仓库表

```sql
CREATE TABLE Warehouses
(
    Code     INTEGER PRIMARY KEY NOT NULL,
    Location TEXT                NOT NULL, -- 仓库地址
    Capacity INTEGER             NOT NULL  -- 容纳箱子的数量
);
```

- 箱子表

```sql
CREATE TABLE Boxes
(
    Code      TEXT PRIMARY KEY NOT NULL,
    Contents  TEXT             NOT NULL, -- 内容
    Value     REAL             NOT NULL, -- 价值
    Warehouse INTEGER          NOT NULL, -- 属于那个仓库
    CONSTRAINT fk_Warehouses_Code FOREIGN KEY (Warehouse) REFERENCES Warehouses (Code)
);
```

# 预置数据

- 仓库信息

```sql
INSERT INTO Warehouses
VALUES (1, 'Chicago', 3),
       (2, 'Chicago', 4),
       (3, 'New York', 7),
       (4, 'Los Angeles', 2),
       (5, 'San Francisco', 8);
```

- 箱子信息

```sql
INSERT INTO Boxes
VALUES ('0MN7', 'Rocks', 180, 3),
       ('4H8P', 'Rocks', 250, 1),
       ('4RT3', 'Scissors', 190, 4),
       ('7G3H', 'Rocks', 200, 1),
       ('8JN6', 'Papers', 75, 1),
       ('8Y6U', 'Papers', 50, 3),
       ('9J6F', 'Papers', 175, 2),
       ('LL08', 'Rocks', 140, 4),
       ('P0H6', 'Scissors', 125, 1),
       ('P2T6', 'Scissors', 150, 2),
       ('TU55', 'Papers', 90, 5);
```

# 题目

1. 选择所有仓库
2. 选择值大于$ 150的所有箱子
3. 在所有箱子中选择所有不同的内容
4. 选择所有箱子的平均值
5. 选择仓库代码和每个仓库中箱子的平均值
6. 与上一练习相同，但只选择那些平均值大于150的仓库
7. 选择每个箱子的代码以及箱子所在城市的名称
8. 选择仓库代码以及每个仓库中的箱子数量。可选，考虑到一些仓库是空的（即，箱子数应该显示为零，而不是从结果中省略仓库）
9. 选择饱和的所有仓库的代码（如果仓库中的箱子数大于仓库的容量，则仓库已饱和）
10. 选择位于芝加哥的所有箱子的代码
11. 在`New York`新建一个仓库，可容纳3个箱子
12. 创建一个新箱子，代码为`H5RT`，包含价值为`200`的`Papers`，位于仓库`2`中
13. 将所有箱子的价值减少15％
14. 对价值大于所有箱子平均值的箱子应用20％的减值
15. 移除价值低于100的所有箱子
16. 从饱和仓库中取出所有箱子

# 答案

1. 选择所有仓库

```sql
select *
from warehouses;
```

2. 选择值大于$ 150的所有箱子

```sql
SELECT *
FROM Boxes
WHERE Value > 150;
```

3. 在所有箱子中选择所有不同的内容

```sql
SELECT DISTINCT Contents
FROM Boxes;
```

4. 选择所有箱子的平均值

```sql
SELECT AVG(Value)
FROM Boxes;
```

5. 选择仓库代码和每个仓库中箱子的平均值

```sql
SELECT Warehouse, AVG(Value)
FROM Boxes
GROUP BY Warehouse;
```

6. 与上一练习相同，但只选择那些平均值大于150的仓库

```sql
SELECT Warehouse, AVG(Value)
FROM Boxes
GROUP BY Warehouse
HAVING AVG(Value) > 150;
```

7. 选择每个箱子的代码以及箱子所在城市的名称

```sql
SELECT Boxes.Code, Location
FROM Warehouses
         INNER JOIN Boxes
                    ON Warehouses.Code = Boxes.Warehouse;
```

8. 选择仓库代码以及每个仓库中的箱子数量。可选，考虑到一些仓库是空的（即，箱子数应该显示为零，而不是从结果中省略仓库）

```sql
/* Not taking into account empty warehouses */
SELECT Warehouse, COUNT(*)
FROM Boxes
GROUP BY Warehouse;

/* Taking into account empty warehouses */
SELECT Warehouses.Code, COUNT(Boxes.Code)
FROM Warehouses
         LEFT JOIN Boxes
                   ON Warehouses.Code = Boxes.Warehouse
GROUP BY Warehouses.Code;
```

9. 选择饱和的所有仓库的代码（如果仓库中的箱子数大于仓库的容量，则仓库已饱和）

```sql
SELECT Code
FROM Warehouses
WHERE Capacity <
      (
          SELECT COUNT(*)
          FROM Boxes
          WHERE Warehouse = Warehouses.Code
      );

/* Alternate method not involving nested statements */
SELECT Warehouses.Code
FROM Warehouses
         JOIN Boxes ON Warehouses.Code = Boxes.Warehouse
GROUP BY Warehouses.Code
HAVING Count(Boxes.Code) > Warehouses.Capacity;
```

10. 选择位于芝加哥的所有箱子的代码

```sql
/* Without subqueries */
SELECT Boxes.Code
FROM Warehouses
         RIGHT JOIN Boxes
                    ON Warehouses.Code = Boxes.Warehouse
WHERE Location = 'Chicago';

/* With a subquery */
SELECT Code
FROM Boxes
WHERE Warehouse IN
      (
          SELECT Code
          FROM Warehouses
          WHERE Location = 'Chicago'
      );
```

11. 在`New York`新建一个仓库，可容纳3个箱子

```sql
INSERT
INTO Warehouses
VALUES (6, 'New York', 3);
```

12. 创建一个新箱子，代码为`H5RT`，包含价值为`200`的`Papers`，位于仓库`2`中

```sql
INSERT INTO Boxes
VALUES ('H5RT', 'Papers', 200, 2);
```

13. 将所有箱子的价值减少15％

```sql
UPDATE Boxes
SET Value = Value * 0.85;
```

14. 对价值大于所有箱子平均值的箱子应用20％的减值

```sql
UPDATE Boxes
SET Value = Value * 0.80
WHERE Value > (SELECT AVG(Value) FROM (SELECT * FROM Boxes) AS X);
```

15. 移除价值低于100的所有箱子

```sql
DELETE
FROM Boxes
WHERE Value < 100;
```

16. 从饱和仓库中取出所有箱子

```sql
DELETE
FROM Boxes
WHERE Warehouse IN
      (
          SELECT Code
          FROM Warehouses
          WHERE Capacity <
                (
                    SELECT COUNT(*)
                    FROM Boxes
                    WHERE Warehouse = Warehouses.Code
                )
      );
```