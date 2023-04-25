# Python获取当前时间戳以及转换

如果要在`Python`中获取时间戳，可以使用`time`、`datetime`模块。

## time

- 获取时间戳

```python
>>> import time
>>> ts1 = time.time()
>>> ts1
1557905910.63995
```

- 反转

```python
>>> readable = time.ctime(ts1)
>>> readable
'Wed May 15 15:38:30 2019'
```

## datetime

- 获取时间戳

```python
>>> import datetime
>>> ts2 = datetime.datetime.now().timestamp()
>>> ts2
1557906112.26428
```

- 反转

```python
>>> readable = datetime.datetime.fromtimestamp(ts2)
>>> readable
datetime.datetime(2019, 5, 15, 15, 41, 52, 264280)
>>> readable.strftime('%Y-%m-%d %H:%M:%S')
'2019-05-15 15:41:52'
```