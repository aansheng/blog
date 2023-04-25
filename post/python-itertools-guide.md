# Python itertools指南

那些`迭代`比你想象中的更为强大。

## 什么是迭代？

简单的说，迭代是可以被`for`循环使用的数据类型，Python中常见的迭代器是列表。

```python
colors = ['red', 'orange', 'yellow', 'green']
```

在上面的例子中，我们创建了一个字符串列表，我们已经给这个列表命名为`colors`。

我们可以使用`for`循环来迭代，下面的的列表中将输出列表中的每一个元素。

```python
for color in colors:
    print(color)
```

Python中有许多不同种类的迭代，但是在本教程中，我们将使用列表。

## 要求

我们必须导入该`itertools`模块才能使用它，我们还将导入`operator`模块。

以下所有示例将包含这些导入。

```python
import itertools
import operator
```

## itertools

### accumulate()

```python
itertools.accumulate（iterable [，func]）
```

该函数会返回函数结果的迭代器，函数可以是变量来传递。`accumulate()`函数将一个函数作为参数。它也需要一个迭代。它返回所有的结果，结果本身包含在一个迭代器中。

- Example

```python
data = [1, 2, 3, 4, 5]

result = itertools.accumulate(data, operator.mul)
for each in result:
    print(each)
```

- Example

```
1
2
6
24
120
```

`operator.mul`需要两个数字并乘以它们

```python
>>> import operator
>>> operator.mul(1, 2)
2
>>> operator.mul(2, 3)
6
>>> operator.mul(6, 4)
24
>>> operator.mul(24, 5)
120
```

在下一个例子中将会使用该`max`功能

- Example

```bash
data = [5, 2, 6, 4, 5, 9, 1]

result = itertools.accumulate(data, max)
for each in result:
    print(each)
```

- Example

```
5 
5 
6 
6 
6 
9 
9
```

该`max`函数返回最大的项

```python
>>> max(5, 2)
5
>>> max(5, 6)
6
>>> max(6, 4)
6
>>> max(6, 5)
6
>>> max(6, 9)
9
>>> max(9, 1)
9
```

传递函数是可选的

- Example

```python
data = [5, 2, 6, 4, 5, 9, 1]

result = itertools.accumulate(data)
for each in result:
    print(each)
```

- Output

```
5 
7 
13 
17 
22 
31 
32
```

如果没有指定功能，项目将相加。

```
5 
5 + 2 = 7 
7 + 6 = 13 
13 + 4 = 17 
17 + 5 = 22 
22 + 9 = 31 
31 + 1 = 32
```

### count()

```python
itertools.count(start=0, step=1)
```

迭代器每次返回`start+step`的值

- Example

返回`1-10`之间的所有奇数

```python
for i in itertools.count(1, 2):
    if i > 10:
        break
    else:
        print(i)
```

- Output

```
1
3
5
7
9
```

### cycle()

```python
itertools.cycle(iterable)
```

无限循环迭代器中的每一个元素

- Example

```python
colors = ['red', 'orange', 'yellow', 'green', 'blue', 'violet']

for color in itertools.cycle(colors):
    print(color)
```

在上面的代码中，我们创建一个列表，然后我们循环或循环遍历这个列表。通常，一个`for`循环逐步循环，直到它到达结束。如果一个列表有`3`个项目，循环将重复3次。但是如果我们使用这个`cycle()`功能的话。当我们到达迭代的结束时，我们从一开始就重新开始。

- Output

```
red
orange
yellow
green
blue
indigo
violet
red
orange
yellow
green
...
```

### repeat()

```python
itertools.repeat(object[, times])
```

此功能将一遍又一遍地重复一个对象，除非有一个`times`次数。

- Example

```python
for i in itertools.repeat("spam"):
    print(i)
```

在上面的代码中，我们创建一个可重复的迭代`spam`，它会不停地循环输出`spam`

- Output

```
spam
spam
spam
spam
spam
spam
...
```

- Example

```python
for i in itertools.repeat("spam", 3):
    print(i)
```

如果我们使用`times`参数，可以限制它将重复的次数。

- Output

```
spam
spam
spam
```

在这个例子中，`spam`只重复三次

### chain()

```python
itertools.chain(*iterables)
```

此函数需要一系列迭代，并将其返回为一个长的迭代。

- Example

```python
colors = ['red', 'orange', 'yellow', 'green', 'blue']
shapes = ['circle', 'triangle', 'square', 'pentagon']

result = itertools.chain(colors, shapes)
for each in result:
    print(each)
```

- Output

```
red
orange
yellow
green
blue
circle
triangle
square
pentagon
```

### compress()

```python
itertools.compress(data, selectors)
```

这个函数可以使用另一个过滤器来迭代

- Example

```python
shapes = ['circle', 'triangle', 'square', 'pentagon']
selections = [True, False, True, False]

result = itertools.compress(shapes, selections)
for each in result:
    print(each)
```

- Output

```
circle
square
```

### dropwhile()

```python
itertools.dropwhile(predicate, iterable)
```

做一个迭代器，只要返回为`true`，就从`iterable`中删除元素，否则就返回后面的每个元素

- Example

```python
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1]

result = itertools.dropwhile(lambda x: x < 5, data)
for each in result:
    print(each)
```

- Output

```
5
6
7
8
9
10
1
```

好。这可以令人困惑 代码说，当项目小于5时，删除每个项目。遇到不少于5的项目后，返回剩下的项目。这就是为什么最后一个被归还。

- Step Through It

```
1 < 5:  True,  drop
2 < 5:  True,  drop
3 < 5:  True,  drop
4 < 5:  True,  drop
5 < 5:  False, return surviving items
```

### groupby()

```python
itertools.groupby(iterable, key=None)
```

简单地说，这个功能将事情集中在一起

- Example

```python

robots = [
    {
        'name': 'blaster',
        'faction': 'autobot'
    },
    {
        'name': 'galvatron',
        'faction': 'decepticon'
    },
    {
        'name': 'jazz',
        'faction': 'autobot'
    },
    {
        'name': 'metroplex',
        'faction': 'autobot'
    },
    {
        'name': 'megatron',
        'faction': 'decepticon'
    },
    {
        'name': 'starcream',
        'faction': 'decepticon'
    }
]

for key, group in itertools.groupby(robots, key=lambda x: x['faction']):
    print(key)
    print(list(group))
```

- Output

```
autobot
[{'name': 'blaster', 'faction': 'autobot'}]
decepticon
[{'name': 'galvatron', 'faction': 'decepticon'}]
autobot
[{'name': 'jazz', 'faction': 'autobot'}, {'name': 'metroplex', 'faction': 'autobot'}]
decepticon
[{'name': 'megatron', 'faction': 'decepticon'}, {'name': 'starcream', 'faction': 'decepticon'}]
```

### filterfalse()

```python
itertools.filterfalse(predicate, iterable)
```

这个函数使迭代器从`iterable`中过滤元素，只返回的元素`False`

- Example

```python
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1]

result = itertools.filterfalse(lambda x: x < 5, data)
for each in result:
    print(each)
```

- Output

```
5
6
7
8
9
10
```

- Debug

```
1 < 5:  True,  drop
2 < 5:  True,  drop
3 < 5:  True,  drop
4 < 5:  True,  drop
5 < 5:  False, keep
6 < 5:  False, keep
7 < 5:  False, keep
8 < 5:  False, keep
9 < 5:  False, keep
10 < 5:  False, keep
1 < 5:  True,  drop
```

### islice()

```python
itertools.islice(iterable, start, stop[, step])
```

这个功能非常像切片，此功能允许您剪切一个可迭代的片段

- Example

```python
colors = ['red', 'orange', 'yellow', 'green', 'blue']
few_colors = itertools.islice(colors, 2)

for each in few_colors:
    print(each)
```

- Output

```
red
orange
```

### starmap()

```python
itertools.starmap(function, iterable)
```

此函数使迭代器使用从`iterable`获取的参数来计算函数

- Example

```python
data = [(2, 6), (8, 4), (7, 3)]

result = itertools.starmap(operator.mul, data)
for each in result:
    print(each)
```

- Output

```
12
32
21
```

- Step Through

```python
>>> operator.mul(2, 6)
12
>>> operator.mul(8, 4)
32
>>> operator.mul(7, 3)
21
```

### tee()

```python
itertools.tee(iterable, n=2)
```

从单个迭代中返回`n`个独立迭代器

- Example

```python
colors = ['red', 'orange', 'yellow', 'green', 'blue']
alpha_colors, beta_colors = itertools.tee(colors)

for each in alpha_colors:
    print(each)

print('..')

for each in beta_colors:
    print(each)
```

默认值为2，但您可以根据需要进行许多操作。

- Output

```
red
orange
yellow
green
blue
..
red
orange
yellow
green
blue
```

### takewhile()

```python
itertools.takwwhile(predicate, iterable)
```

这是相反的`dropwhile()`，只要返回为`true`，该函数就可以使用迭代器并从`iterable`返回元素

- Example

```python
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1]
result = itertools.takewhile(lambda x: x < 5, data)

for each in result:
    print(each)
```

- Output

```
1
2
3
4
```

- Step Through It

```
1 < 5:  True,  keep going
2 < 5:  True,   keep going
3 < 5:  True,   keep going
4 < 5:  True,   keep going
5 < 5:  False, stop and drop
```

### zip_longest()

```python
itertools.zip_longest(*iterables, fillvalue=None)
```

此函数使迭代器聚合每个迭代的元素，如果迭代长度不均匀，则缺少的值将被填充为`fillvalue`。迭代继续，直到最长的迭代耗尽。

- Example

```python
colors = ['red', 'orange', 'yellow', 'green', 'blue']
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

for each in itertools.zip_longest(colors, data, fillvalue=None):
    print(each)
```

- Output

```
('red', 1)
('orange', 2)
('yellow', 3)
('green', 4)
('blue', 5)
(None, 6)
(None, 7)
(None, 8)
(None, 9)
(None, 10)
```

### product()

此函数从一系列迭代创建[笛卡尔乘积](https://en.wikipedia.org/wiki/Cartesian_product)。

- Example

```python
num_data = [1, 2, 3]
alpha_data = ['a', 'b', 'c']
result = itertools.product(num_data, alpha_data)

for each in result:
    print(each)
```

- Output

```
(1, 'a')
(1, 'b')
(1, 'c')
(2, 'a')
(2, 'b')
(2, 'c')
(3, 'a')
(3, 'b')
(3, 'c')
```

想象一下这样的桌子：

```
      a     b      c
1     a1    b1     c1
2     a2    b2     c3
3     a3    b3     b3
```

### permutations()

```python
itertools.permutations(iterable, r=None)
```

- Example

```python
alpha_data = ['a', 'b', 'c']
result = itertools.permutations(alpha_data)

for each in result:
    print(each)
```

- Output

```
('a', 'b', 'c')
('a', 'c', 'b')
('b', 'a', 'c')
('b', 'c', 'a')
('c', 'a', 'b')
('c', 'b', 'a')
```

### combinations()

```python
itertools.combinations(iterable, r)
```

此函数需要一个迭代和一个整数，这将创建具有`r`成员的所有独特组合。

- Example

```python
shapes = ['circle', 'triangle', 'square', ]
result = itertools.combinations(shapes, 2)

for each in result:
    print(each)
```

在这段代码中，我们使用2个成员组合所有组合。

- Output

```
（'circle'，'triangle'）
（'circle'，'square'）
（'triangle'，'square'）
```

- Example

```python
result = itertools.combinations(shapes, 3)
for each in result:
    print(each)
```

在这段代码中，我们使用3个成员组合所有组合。这有点不太令人兴奋。

- Output

```
('circle', 'triangle', 'square')
```

### combinations_with_replacement()

```python
itertools.combinations_with_replacement(iterable, r)
```

这一个就像`combinations()`功能一样，但是这个可以让单个元素重复一次。

- Example

```python
shapes = ['circle', 'triangle', 'square', ]

result = itertools.combinations_with_replacement(shapes, 2)
for each in result:
    print(each)
```

- Output

```
('circle', 'circle')
('circle', 'triangle')
('circle', 'square')
('triangle', 'triangle')
('triangle', 'square')
('square', 'square')
```
