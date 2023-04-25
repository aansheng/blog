# 使用Python的30个技巧

## Tips#1. 变量交换

- Example

```python
x, y = 10, 20
print(x, y)

x, y = y, x
print(x, y)
```

- Output

```
10 20
20 10
```

## Tips#2. 链接比较运算符

有时候比较运算符的聚合是另一个可以方便的手段

- Example

```python
n = 10
result = 1 < n < 20
print(result)

result = 1 > n <= 9
print(result)
```

- Output

```
True
False
```

## Tips#3. 使用三元运算符进行条件判断

三元运算符是if-else语句的快捷方式，也称为条件运算符

```python
[on_true] if [expression] else [on_false]
```

以下是您可以使用的一些示例，使您的代码紧凑简洁

- Example

如果`y`为`9`，则将`10`分配给`x`，否则将`20`分配给`x`

```python
x = 10 if (y == 9) else 20
```

同样，我们可以对类对象做同样的事情

```python
x = (classA if y == 1 else classB)(param1, param2)
```

在上面的例子中，`classA`和`classB`是两个类，一个类构造函数将被调用。

- Example

算出最小的数量

```python
def small(a, b, c):
    return a if a <= b and a <= c else(b if b <= a and b <= c else c)


print(small(1, 0, 1))
print(small(1, 2, 2))
print(small(2, 2, 3))
print(small(5, 4, 3))
```

- Output

```
0
1
2
3
```

我们甚至可以使用具有列表理解的三元运算符

```python
>>> [m**2 if m > 10 else m**4 for m in range(10)]
[0, 1, 16, 81, 256, 625, 1296, 2401, 4096, 6561]
```

## Tips#4. 使用多行字符串

基本的方法是使用C语言导出的`反斜杠`

- Example

```python
>>> "select * from multi_row \
... where row_id < 5"
'select * from multi_row where row_id < 5'
```

另外一个诀窍就是使用三个引号

- Example

```python
>>> """select * from multi_row 
... where row_id < 5"""
'select * from multi_row \nwhere row_id < 5'
```

又或者你可以将字符串放在括号中

- Example

```python
>>> ("select * from multi_row "
... "where row_id < 5 "
... "order by age") 
'select * from multi_row where row_id < 5 order by age'
```

## Tips#5. 将列表的元素存储到新变量中

我们可以使用一个列表来初始化一个`no`的变量。在拆包列表时，变量的数量不应超过`no`的列表中的元素。

- Example

```python
testList = [1, 2, 3]
x, y, z = testList

print(x, y, z)
```

- Output

```
1 2 3
```

## Tips#6. 输出导入模块的文件路径

如果您想知道在代码中导入模块的绝对位置，则使用以下技巧

- Example

```python
import threading
import socket

print(threading)
print(socket)
```

- Output

```
<module 'threading' from '/home/ansheng/.pyenv/versions/3.6.2/lib/python3.6/threading.py'>
<module 'socket' from '/home/ansheng/.pyenv/versions/3.6.2/lib/python3.6/socket.py'>
```

## Tips#7. 使用交互式`_`运算符

在Python控制台中，每当我们测试一个表达式或者调用一个函数时，结果将分派到一个临时名称`_（一个下划线）`

- Example

```python
>>> 2 + 1
3
>>> _
3
```

> `_`引用最后执行的表达式的输出

## Tips#8. 词典/集合推导

像列表推导一样，字典/集合同样也有推导，它们使用简单，同样有效

- Example

```python
>>> {i: i * i for i in range(10)}
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36, 7: 49, 8: 64, 9: 81}
>>> {i * 2 for i in range(10)}
{0, 2, 4, 6, 8, 10, 12, 14, 16, 18}
```

## Tips#9. 调试脚本

我们可以借助于`pdb`模块在Python脚本中设置断点

- Example

```python
import pdb
pdb.set_trace()
```

## Tips#10. 文件共享

Python允许运行一个HTTP服务器，您可以使用它来从服务器根目录共享文件

- Example

```python
~$ python -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

以上命令将启动默认端口即`8000`的服务器，也可以通过将自定义端口作为上一个参数传递给上述命令

## Tips#11. 检查Python中的对象

我们可以通过调用`dir()`方法来检查Python中的对象。

- Example

```python
>>> test = [1,2,3]
>>> dir(test)
['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
```

## Tips#12. 简化if语句

要验证多个值，我们可以通过以下方式进行。

- Example

```python
if m in [1,3,5,7]:
```

或者，我们可以使用`{1,3,5,7}`而不是`[1,3,5,7]`用于`in`运算符，因为`set`可以通过`O（1）`访问每个元素

## Tips#13. 在运行时检测Python版本

有时如果当前运行的`Python`小于受支持的版本，我们可能不想执行我们的程序，要实现这一点，您可以使用下面的代码片段，它还以可读格式打印当前使用的Python版本。

- Example

```python
>>> import sys
>>> print("Current Python version: ", sys.version)
Current Python version:  3.6.2 (default, Aug 25 2017, 09:52:18) 
[GCC 6.3.0 20170406]
```

```python
>>> if not hasattr(sys, "hexversion") or sys.hexversion != 50660080:
...     print("Sorry, you aren't running on Python 3.5\n")
...     print("Please upgrade to 3.5.\n")
...     sys.exit(1)
... 
Sorry, you aren't running on Python 3.5

Please upgrade to 3.5.
```

> sys.version_info >= (3, 5) sys.hexversion!= 50660080

## Tips#14. 组合多个字符串

如果要连接列表中可用的所有元素，请参见以下示例。

- Example

```python
>>> test = ['I', 'Like', 'Python', 'automation']
```

现在，我们从上面给出的列表中的元素创建一个单一的字符串。

```python
>>> ''.join(test)
'ILikePythonautomation'
```

## Tips#15. 四种方式来反转字符串/列表

- Example

反转列表本身

```python
>>> testList = [1, 3, 5]
>>> testList.reverse()
>>> testList
[5, 3, 1]
```

再循环中反复循环

```python
>>> for element in reversed([1,3,5]): print(element)
... 
5
3
1
```

反转一行字符串

```python
>>> "Test Python"[::-1]
'nohtyP tseT'
```

使用切片翻转列表

```python
>>> [1, 3, 5][::-1]
[5, 3, 1]
```

## Tips#16. 枚举

使用枚举器，在循环中找到索引很容易

- Example

```python
>>> testlist = [10, 20, 30]
>>> for i, value in enumerate(testlist):
...     print(i, ': ', value)
... 
0 :  10
1 :  20
2 :  30
```

## Tips#17. 在Python中使用枚举

我们可以使用以下方法来创建枚举定义

- Example

```python
>>> class Shapes:
...     Circle, Square, Triangle, Quadrangle = range(4)
... 
>>> Shapes.Circle
0
>>> Shapes.Square
1
>>> Shapes.Triangle
2
>>> Shapes.Quadrangle
3
```

## Tips#18. 从函数中返回多个值

- Example

```python
def x():
    return 1, 2, 3, 4


a, b, c, d = x()

print(a, b, c, d)
```

- Output

```
1 2 3 4
```

## Tips#19. 使用Splat运算符解包函数参数

- Example

```python
def test(x, y, z):
    print(x, y, z)


testDict = {'x': 1, 'y': 2, 'z': 3}
testList = [10, 20, 30]

test(*testDict)
test(**testDict)
test(*testList)
```

- Output

```
x y z
1 2 3
10 20 30
```

## Tips#20. 使用字典存储开关

- Example

```python
stdcalc = {
    'sum': lambda x, y: x + y,
    'subtract': lambda x, y: x - y
}

print(stdcalc['sum'](9, 3))
print(stdcalc['subtract'](9, 3))
```

- Output

```
12
6
```

## Tips#21. 计算一行中任何数字的阶乘

- Example

```python
>>> import functools
>>> result = (lambda k: functools.reduce(int.__mul__, range(1,k+1),1))(3)
>>> result
6
```

## Tips#22. 查找列表中最常见的值

- Example

```python
>>> test = [1,2,3,4,2,2,3,1,4,4,4]
>>> print(max(set(test), key=test.count))
4
```

## Tips#23. 重置递归限制

Python将递归限制限制为`1000`，我们可以重置其值

- Example

```python
>>> import sys
>>> x = 1001
>>> print(sys.getrecursionlimit())
1000
>>> sys.setrecursionlimit(x)
>>> print(sys.getrecursionlimit())
1001
```

## Tips#24. 检查对象的内存使用情况

在`Python 3.5`中`32`位整数使用`28字节`，要验证内存使用情况，我们可以调用`<getsizeof>`方法

- Example

```python
>>> import sys
>>> x=1
>>> print(sys.getsizeof(x))
28
```

## Tips#25. 使用\__slots__来减少内存开销


你有没有观察过你的Python应用程序消耗大量资源，特别是内存？

这是使用`<__ slots __>`类变量在一定程度上减少内存开销的一个技巧

- Example

```python
class FileSystem(object):
    def __init__(self, files, folders, devices):
        self.files = files
        self.folders = folders
        self.devices = devices


print(sys.getsizeof(FileSystem))


class FileSystem1(object):
    __slots__ = ['files', 'folders', 'devices']

    def __init__(self, files, folders, devices):
        self.files = files
        self.folders = folders
        self.devices = devices


print(sys.getsizeof(FileSystem1))
```

- Output

```
1056
888
```

## Tips#26. Lambda模仿打印功能

- Example

```python
>>> import sys
>>> lprint=lambda *args:sys.stdout.write(" ".join(map(str,args)))
>>> lprint("python", "tips",1000,1001)
python tips 1000 100121
```

## Tips#27. 从两个序列创建一个字典

- Example

```python
>>> t1 = (1, 2, 3)
>>> t2 = (10, 20, 30)
>>> print(dict (zip(t1,t2)))
{1: 10, 2: 20, 3: 30}
```

## Tips#28. 在字符串中搜索多个前缀

- Example

```python
>>> print("http://www.google.com".startswith(("http://", "https://")))
True
>>> print("http://www.google.co.uk".endswith((".com", ".co.uk")))
True
```

## Tips#29. 形成统一列表，而不使用任何循环

- Example

```python
>>> import itertools
>>> test = [[-1, -2], [30, 40], [25, 35]]
>>> print(list(itertools.chain.from_iterable(test)))
[-1, -2, 30, 40, 25, 35]
```

如果您有一个包含嵌套列表或元组的输入列表作为元素，那么请使用以下技巧。然而，这里的限制是它使用一个for循环。

- Example

```python
def unifylist(l_input, l_target):
    for it in l_input:
        if isinstance(it, list):
            unifylist(it, l_target)
        elif isinstance(it, tuple):
            unifylist(list(it), l_target)
        else:
            l_target.append(it)
    return l_target


test = [[-1, -2], [1, 2, 3, [4, (5, [6, 7])]], (30, 40), [25, 35]]

print(unifylist(test, []))
```

- Output

```
[-1, -2, 1, 2, 3, 4, 5, 6, 7, 30, 40, 25, 35]
```

统一包含列表和元组的列表的另一个更简单的方法是使用Python的`more_itertools`包。

它不需要循环。只要做一个`pip install more_itertools`，如果还没有

- Example

```python
>>> import more_itertools
>>> test = [[-1, -2], [1, 2, 3, [4, (5, [6, 7])]], (30, 40), [25, 35]]
>>> print(list(more_itertools.collapse(test)))
[-1, -2, 1, 2, 3, 4, 5, 6, 7, 30, 40, 25, 35]
```

## Tips#30. 在Python中实现真正的Switch-Case语句

以下是使用字典来模拟开关案例构造的代码

- Example

```python
def xswitch(x):
    return xswitch._system_dict.get(x, None)


xswitch._system_dict = {'files': 10, 'folders': 5, 'devices': 2}

print(xswitch('default'))
print(xswitch('devices'))
```

- Output

```
None
2
```
