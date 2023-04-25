# SQL语句练习-员工管理

# 创建表

- 部门表

```sql
CREATE TABLE departments
(
    Id     INTEGER PRIMARY KEY NOT NULL, -- 部门ID
    Name   varchar(100),                 -- 部门名称
    Budget FLOAT                         -- 部门预算
);
```

- 员工表

```sql
CREATE TABLE employees
(
    SSN        INTEGER PRIMARY KEY NOT NULL,
    Name       varchar(100)        NOT NULL,
    LastName   varchar(100)        NOT NULL,
    Department INTEGER             NOT NULL,
    CONSTRAINT fk_Departments_Code FOREIGN KEY (Department)
        REFERENCES Departments (Id)
);
```

# 预置数据

- 部门信息

```sql
INSERT INTO departments
VALUES (14, 'IT', 65000),
       (37, '财务', 15000),
       (59, '人力资源', 240000),
       (77, '研发', 55000);
```

- 员工信息

```sql
INSERT INTO Employees
VALUES ('123234877', 'Michael', 'Rogers', 14),
       ('152934485', 'Anand', 'Manikutty', 14),
       ('222364883', 'Carol', 'Smith', 37),
       ('326587417', 'Joe', 'Stevens', 37),
       ('332154719', 'Mary-Anne', 'Foster', 14),
       ('332569843', 'George', 'O''Donnell', 77),
       ('546523478', 'John', 'Doe', 59),
       ('631231482', 'David', 'Smith', 77),
       ('654873219', 'Zacary', 'Efron', 59),
       ('745685214', 'Eric', 'Goldsmith', 59),
       ('845657245', 'Elizabeth', 'Doe', 14),
       ('845657246', 'Kumar', 'Swamy', 14);
```

# 题目

1. 选择所有员工的姓氏
2. 选择所有员工的姓氏，不重复
3. 选择姓氏为`Smith`的员工的所有数据
4. 选择姓氏为`Smith`或`Doe`的员工的所有数据
5. 选择在部门`14`中工作的员工的所有数据
6. 选择在部门`37`或部门`77`中工作的员工的所有数据
7. 选择姓氏以`S`开头的员工的所有数据
8. 选择所有部门预算的总和
9. 选择每个部门的员工数量（只需要显示部门ID和员工数量）
10. 选择员工的所有数据，包括每个员工的部门数据
11. 选择每个员工的姓名和姓氏，以及员工部门的名称和预算
12. 选择为预算超过`60,000`的部门工作的员工的姓名和姓氏
13. 选择预算大于所有部门平均预算的部门
14. 选择拥有两名以上员工的部门的名称
15. 选择为预算第二低的部门工作的员工的姓名和姓氏
16. 添加一个名为`Quality Assurance`的新部门，预算为`40,000`美元，部门代码为`11`，在该部门添加一名名为`Mary Moore`的员工，SSN为`847-21-9811`
17. 将所有部门的预算减少`10％`
18. 将所有员工从研究部门（代码77）重新分配给IT部门（代码14）
19. 从表中删除IT部门的所有员工（代码14）
20. 从表中删除所有在预算大于或等于`60,000`的部门工作的员工
21. 从表中删除所有员工

# 答案

1. 选择所有员工的姓氏

```sql
select lastname
from employees;
```

2. 选择所有员工的姓氏，不重复

```sql
select distinct lastname
from employees;
```

3. 选择姓氏为`Smith`的员工的所有数据

```sql
select *
from employees
where lastname = 'Smith';
```

4. 选择姓氏为`Smith`或`Doe`的员工的所有数据

```sql
/* With OR */
SELECT *
FROM Employees
WHERE LastName = 'Smith'
   OR LastName = 'Doe';

/* With IN */
SELECT *
FROM Employees
WHERE LastName IN ('Smith', 'Doe');
```

5. 选择在部门`14`中工作的员工的所有数据

```sql
select *
from employees
where department = 14;
```

6. 选择在部门`37`或部门`77`中工作的员工的所有数据

```sql
/* With OR */
select *
from employees
where department = 37
   or department = 77;

/* With IN */
select *
from employees
where department in (37, 77);
```

7. 选择姓氏以`S`开头的员工的所有数据

```sql
SELECT *
FROM Employees
WHERE LastName LIKE 'S%';
```

8. 选择所有部门预算的总和

```sql
select sum(budget)
from departments;
```

9. 选择每个部门的员工数量（只需要显示部门ID和员工数量）

```sql
select department, count(*)
from employees
group by department;
```

10. 选择员工的所有数据，包括每个员工的部门数据

```sql
SELECT SSN, E.Name AS Name_E, LastName, D.Name AS Name_D, Department, id, Budget
FROM Employees E
         INNER JOIN Departments D
                    ON E.Department = D.id;
```

11. 选择每个员工的姓名和姓氏，以及员工部门的名称和预算

```sql
/* Without labels */
SELECT Employees.Name, LastName, Departments.Name AS DepartmentsName, Budget
FROM Employees
         INNER JOIN Departments
                    ON Employees.Department = Departments.ID;

/* With labels */
SELECT E.Name, LastName, D.Name AS DepartmentsName, Budget
FROM Employees E
         INNER JOIN Departments D
                    ON E.Department = D.ID;
```

12. 选择为预算超过`60,000`的部门工作的员工的姓名和姓氏

```sql
/* Without subquery */
SELECT Employees.Name, LastName
FROM Employees
         INNER JOIN Departments
                    ON Employees.Department = Departments.id
                        AND Departments.Budget > 60000;

/* With subquery */
SELECT Name, LastName
FROM Employees
WHERE Department IN
      (SELECT id FROM Departments WHERE Budget > 60000);
```

13. 选择预算大于所有部门平均预算的部门

```sql
SELECT *
FROM Departments
WHERE Budget >
      (
          SELECT AVG(Budget)
          FROM Departments
      );
```

14. 选择拥有两名以上员工的部门的名称

```sql
/* With subquery */
SELECT Name
FROM Departments
WHERE id IN
      (
          SELECT Department
          FROM Employees
          GROUP BY Department
          HAVING COUNT(*) > 2
      );

/* With UNION. This assumes that no two departments have the same name */
SELECT Departments.Name
FROM Employees
         INNER JOIN Departments
                    ON Department = id
GROUP BY Departments.Name
HAVING COUNT(*) > 2;
```

15. 选择为预算第二低的部门工作的员工的姓名和姓氏

```sql
select name, lastname
from employees
where department = (
    select id
    from departments
    order by budget
    limit 1 offset 1);
```

16. 添加一个名为`Quality Assurance`的新部门，预算为`40,000`美元，部门代码为`11`，在该部门添加一名名为`Mary Moore`的员工，SSN为`847-21-9811`

```sql
INSERT INTO Departments
VALUES (11, 'Quality Assurance', 40000);

INSERT INTO Employees
VALUES ('847219811', 'Mary', 'Moore', 11);
```

17. 将所有部门的预算减少`10％`

```sql
UPDATE Departments
SET Budget = Budget * 0.9;
```

18. 将所有员工从研究部门（代码77）重新分配给IT部门（代码14）

```sql
UPDATE Employees
SET Department = 14
WHERE Department = 77;
```

19. 从表中删除IT部门的所有员工（代码14）

```sql
DELETE
FROM Employees
WHERE Department = 14;
```

20. 从表中删除所有在预算大于或等于`60,000`的部门工作的员工

```sql
DELETE
FROM Employees
WHERE Department IN
      (
          SELECT id
          FROM Departments
          WHERE Budget >= 60000
      );
```

21. 从表中删除所有员工

```sql
DELETE
FROM Employees;
```