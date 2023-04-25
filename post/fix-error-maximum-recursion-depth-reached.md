# 修复Python递归超出最大深度

刚用Python写了一个递归小程序，功能主要是像`函数(print_n(n))`传入一个`n`值，然后输出`1-n`，例如下面的代码：

```python
def print_n(n):
    if n:
        print_n(n - 1)
    print(n)
```

当我在`ipython`中进行调试的时候，N为`10000`时，会抛出`RecursionError: maximum recursion depth exceeded`的错误

```python
In [1]: def print_n(n):
   ...:     if n:
   ...:         print_n(n - 1)
   ...:     print(n)
   ...:

In [2]: print_n(10000)
---------------------------------------------------------------------------
RecursionError                            Traceback (most recent call last)
<ipython-input-2-085ca2d6a389> in <module>
----> 1 print_n(10000)

<ipython-input-1-ae5822a69b3f> in print_n(n)
      1 def print_n(n):
      2     if n:
----> 3         print_n(n - 1)
      4     print(n)
      5

... last 1 frames repeated, from the frame below ...

<ipython-input-1-ae5822a69b3f> in print_n(n)
      1 def print_n(n):
      2     if n:
----> 3         print_n(n - 1)
      4     print(n)
      5

RecursionError: maximum recursion depth exceeded
```

这其实是因为Python限制了递归的深度，可以用过`sys.getrecursionlimit()`查看限制的深度

```python
In [3]: import sys

In [4]: sys.getrecursionlimit()
Out[4]: 3000
```

可以看到，默认的限制是`3000`，当然我们也可以修改这个限制，通过`sys.setrecursionlimit`方法

```python
In [5]: sys.setrecursionlimit(20000)

In [6]: sys.getrecursionlimit()
Out[6]: 20000
```

这会将默认限制增加到`20000`，然后我们再次调用`print_n`函数

```python
In [7]: print_n(10000)
...
10000
```

最终，解决了我们的问题。