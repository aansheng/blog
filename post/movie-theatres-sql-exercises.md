# SQL语句练习-电影院

# 创建表

- 电影表

```sql
CREATE TABLE Movies
(
    Code   INTEGER PRIMARY KEY NOT NULL,
    Title  TEXT                NOT NULL, -- 标题
    Rating TEXT                          --评分
);
```

- 电影院表

```sql
CREATE TABLE MovieTheaters
(
    Code  INTEGER PRIMARY KEY NOT NULL,
    Name  TEXT                NOT NULL,--名称
    Movie INTEGER                      --电影
        CONSTRAINT fk_Movies_Code REFERENCES Movies (Code)
);
```

# 预置数据

- 电影信息

```sql
INSERT INTO Movies
VALUES (9, 'Citizen King', 'G'),
       (1, 'Citizen Kane', 'PG'),
       (2, 'Singin'' in the Rain', 'G'),
       (3, 'The Wizard of Oz', 'G'),
       (4, 'The Quiet Man', NULL),
       (5, 'North by Northwest', NULL),
       (6, 'The Last Tango in Paris', 'NC-17'),
       (7, 'Some Like it Hot', 'PG-13'),
       (8, 'A Night at the Opera', NULL);
```

- 影院信息

```sql
INSERT INTO MovieTheaters
VALUES (1, 'Odeon', 5),
       (2, 'Imperial', 1),
       (3, 'Majestic', NULL),
       (4, 'Royale', 6),
       (5, 'Paraiso', 3),
       (6, 'Nickelodeon', NULL);
```

# 题目

1. 选择所有电影的标题
2. 显示数据库中的所有不同评级
3. 显示所有未评级的电影
4. 选择当前未显示电影的所有电影院
5. 选择所有电影院的所有数据，以及剧院中正在显示的电影中的数据（如果正在显示）
6. 从所有电影中选择所有数据，如果该影片正在影院中显示，则显示影院中的数据
7. 显示当前未在任何影院放映的影片的标题
8. 添加未评级的电影`One, Two, Three`
9. 将所有未评级影片的等级设为`G`
10. 删除投影电影`NC-17`的电影院

# 答案

1. 选择所有电影的标题

```sql
SELECT Title
FROM Movies;
```

2. 显示数据库中的所有不同评级

```sql
SELECT DISTINCT Rating
FROM Movies;
```

3. 显示所有未评级的电影

```sql
SELECT *
FROM Movies
WHERE Rating IS NULL;
```

4. 选择当前未显示电影的所有电影院

```sql
SELECT *
FROM MovieTheaters
WHERE Movie IS NULL;
```

5. 选择所有电影院的所有数据，以及剧院中正在显示的电影中的数据（如果正在显示）

```sql
SELECT *
FROM MovieTheaters
         LEFT JOIN Movies
                   ON MovieTheaters.Movie = Movies.Code;
```

6. 从所有电影中选择所有数据，如果该影片正在影院中显示，则显示影院中的数据

```sql
SELECT *
FROM MovieTheaters
         RIGHT JOIN Movies
                    ON MovieTheaters.Movie = Movies.Code;
```

7. 显示当前未在任何影院放映的影片的标题

```sql
/* With JOIN */
SELECT Movies.Title
FROM MovieTheaters
         RIGHT JOIN Movies
                    ON MovieTheaters.Movie = Movies.Code
WHERE MovieTheaters.Movie IS NULL;

/* With subquery */
SELECT Title
FROM Movies
WHERE Code NOT IN
      (
          SELECT Movie
          FROM MovieTheaters
          WHERE Movie IS NOT NULL
      );
```

8. 添加未评级的电影`One, Two, Three`

```sql
INSERT INTO Movies
VALUES (10, 'One, Two, Three', NULL);
```

9. 将所有未评级影片的等级设为`G`

```sql
UPDATE Movies
SET Rating='G'
WHERE Rating IS NULL;
```

10. 删除投影电影`NC-17`的电影院

```sql
DELETE
FROM MovieTheaters
WHERE Movie IN
      (SELECT Code FROM Movies WHERE Rating = 'NC-17');
```