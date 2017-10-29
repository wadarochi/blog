---
title: 怎样写python的if语句比较高效？
date: 2017-10-15 01:49:38
tags:
 - python
---

# 问题来源
跳转，尤其是条件跳转有可能会打断CPU指令的流水线执行，所以在性能要求比较高的代码中，我都是系统尽量不要出现if，那实在要出现怎么办了，比如有如下代码：
```python
for component in components:
    # 判断是否要处理component数据
    if component.enabled:
        # do sth...
```

这种情况下我是应该写成上面这种形式，还是应该写成下面的形式呢，哪种更高效？

```python
for component in components:
    # 判断是否要处理component数据
    if not component.enabled:
        continue
    # do sth...
```

别问我为什么性能要求比较高的时候还要用python，我也不想......那么假设我们知道条件为`True`和`False`的概率，哪一种比较高效？

<!--more-->

# Python解释器中if语句的实现细节
## 字节码
上述的第一种编码方式更容易生成箭头型的代码，直观的解释就是下面这种：
```python
for a in balabala:
    if a.b:
        if a.c:
            if a.d:
                # do sth...
```

所以我们经常使用`Guard Clauses`的方式去规避这样的箭头型代码，看起来会比较漂亮一些，就是下面这种：

```python
for a in balabala:
    if not a.b:
        continue

    if not a.c:
        continue

    if not a.d:
        continue

    # do sth...
```

回到之前的问题，哪种比较高效？首先我们给出示例代码：

```python
In [1]: from dis import dis

In [2]: class Component(object):
   ...:     def __init__(self):
   ...:         self.enabled = True
   ...:     def update(self, delta_time_s):
   ...:         pass
   ...: 

In [3]: components = [Component() for idx in xrange(10000)]

In [4]: def update_in_arrow_form(delta_time_s):
   ...:     for c in components:
   ...:         if c.enabled:
   ...:             c.update(delta_time_s)
   ...: 

In [5]: def update_in_guard_clauses_form(delta_time_s):
   ...:     for c in components:
   ...:         if not c.enabled:
   ...:             continue
   ...:         c.update(delta_time_s)
   ...:
```

在上边的代码中，我们模拟了一个遍历Component容器并且调用每一个Component的update方法(这种Component的设计是很糟糕的，状态和行为还是需要分割开来比较好，参考ECS)，遍历过程中需要判定Component是否需要处理，这就引入了分支逻辑。

在看看字节码之前，我先根据过往教科书上的知识做一个猜想：**引发分支预测错误少、分支预测错误代价小的写法，性能更好**。

然后我们来看看箭头型代码的字节码：

```python
In [6]: dis(update_in_arrow_form)
  2           0 SETUP_LOOP              39 (to 42)
              3 LOAD_GLOBAL              0 (components)
              6 GET_ITER
        >>    7 FOR_ITER                31 (to 41)
             10 STORE_FAST               1 (c)

  3          13 LOAD_FAST                1 (c)
             16 LOAD_ATTR                1 (enabled)
             19 POP_JUMP_IF_FALSE       38

  4          22 LOAD_FAST                1 (c)
             25 LOOKUP_METHOD            2 (update)
             28 LOAD_FAST                0 (delta_time_s)
             31 CALL_METHOD              1
             34 POP_TOP
             35 JUMP_ABSOLUTE            7
        >>   38 JUMP_ABSOLUTE            7
        >>   41 POP_BLOCK
        >>   42 LOAD_CONST               0 (None)
             45 RETURN_VALUE
```

可以看出只有在判断`c.enabled`为`False`的情况下，才需要跳转，假如`c.enabled`大概率为`True`的时候，类似逻辑写成箭头型符合之前的猜想，但是如果`c.enabled`大概率为`False`，就不符合了。

那`Guard Clauses`这种写法呢？也看看字节码：

```python
In [7]: dis(update_in_guard_clauses_form)
  2           0 SETUP_LOOP              42 (to 45)
              3 LOAD_GLOBAL              0 (components)
              6 GET_ITER
        >>    7 FOR_ITER                34 (to 44)
             10 STORE_FAST               1 (c)

  3          13 LOAD_FAST                1 (c)
             16 LOAD_ATTR                1 (enabled)
             19 POP_JUMP_IF_TRUE        28

  4          22 JUMP_ABSOLUTE            7
             25 JUMP_FORWARD             0 (to 28)

  5     >>   28 LOAD_FAST                1 (c)
             31 LOOKUP_METHOD            2 (update)
             34 LOAD_FAST                0 (delta_time_s)
             37 CALL_METHOD              1
             40 POP_TOP
             41 JUMP_ABSOLUTE            7
        >>   44 POP_BLOCK
        >>   45 LOAD_CONST               0 (None)
             48 RETURN_VALUE
```

只要`c.enabled`为`True`，就会引发跳转，那么就和之前的箭头型代码反过来了：假如`c.enabled`大概率为`True`的时候，类似逻辑写成`Guard Clauses`型是不符合猜想的，但是如果`c.enabled`大概率为`False`，就符合猜想。

虽然上面的例子没有加上`else`分支，但是如果你自己加上`else`分支，也是类似的字节码。

## CPython的底层实现

到目前为止，我们看到的都是字节码层面的内容，`CPython`底层`POP_JUMP_IF_TRUE`和`POP_JUMP_IF_FALSE`是如何处理跳转的呢？答案在`ceval.c`中，这里我只贴`POP_JUMP_IF_FALSE`相关的代码，`POP_JUMP_IF_TRUE`其实是类似的：

```c
        PREDICTED(POP_JUMP_IF_FALSE);
        TARGET(POP_JUMP_IF_FALSE) {
            PyObject *cond = POP();
            int err;
            if (cond == Py_True) {
                Py_DECREF(cond);
                FAST_DISPATCH();
            }
            if (cond == Py_False) {
                Py_DECREF(cond);
                JUMPTO(oparg);
                FAST_DISPATCH();
            }
            err = PyObject_IsTrue(cond);
            Py_DECREF(cond);
            if (err > 0)
                ;
            else if (err == 0)
                JUMPTO(oparg);
            else
                goto error;
            DISPATCH();
        }
```

果然，字面意思就是如果判断为`False`我才跳转，`JUMPTO`只有在`cond == Py_False`的时候才会被调用到。

那这么说，是不是意味着：**最可能运行到的代码应该放到if分支中，箭头型代码比Guard Clauses重构过的代码性能要好？**

## Benchmark
如果没有Benchmark，一切都只是猜想，因为一方面CPU的流水线分支预测非常复杂，还有乱序执行等等高级技术在里面；另一方面除了`JUMPTO`，上面解释执行Python代码的C代码中还有大量的`if-else`分支，也就是说只要出现这条字节码，对CPU就会多出非常多的分支。所以单从字节码层面的跳转分析可能是不准的，只好测一测了。

这里我构造两组`components`，第一组只有每十个中只有一个`Component`的`enabled`为`True`，另一组反过来，然后看看运行时间的对比：

```python
In [8]: def test_almost_true(f):
   ...:     for idx, component in enumerate(components):
   ...:         if idx % 10 == 0:
   ...:             component.enabled = False
   ...:         else:
   ...:             component.enabled = True
   ...:     return timeit.repeat(lambda: f(0), number=10)
   ...:

In [9]: def test_almost_false(f):
   ...:     for idx, component in enumerate(components):
   ...:         if idx % 10 == 0:
   ...:             component.enabled = True 
   ...:         else:
   ...:             component.enabled = False
   ...:     return timeit.repeat(lambda: f(0), number=10)
   ...:

In [10]: a1 = test_almost_true(update_in_arrow_form)

In [11]: b1 = test_almost_true(update_in_guard_clauses_form)

In [12]: for m, n in zip(a1, b1):
    ...:     print (m - n) / n * 100.0
    ...:
23.0675013609
-23.0445608311
-16.3107397091
30.4611497157
-2.43304228392
-4.31911240259
79.3438445612

In [13]: def no_branch(delta_time_s):
    ...:     for c in components:
    ...:         c.update(delta_time_s)
    ...:

In [14]: g = test_almost_true(no_branch)

In [15]: for m, n in zip(a1, g):
    ...:     print (m - n) / n * 100
    ...: 
204.991568297
151.134771645
209.973251815
313.04
303.284671533
174.186222559
357.392363932
```

&\*%^)$#，看起来在字节码和c代码层面根据跳转与否做出的猜想，与测试结果不一致...

经过了非常多的测试，看起来在Intel i5平台上（移动版，性能很差，从上面的数据就可以看出来了），有这样的结论：

1. 即使在有先验概率支持的情况下，箭头型代码并不比`Guard Clauses`重构过的代码快，两者的测试看不出显著差别，那么我们还是继续使用`Guard Clauses`形式的代码好了；
2. 相对于没有分支的代码，有分支的代码要慢（相对百分之两三百，不过绝对值非常小），尽管没分支的测试代码多调用了11%的函数(这个和经验倒是吻合的)。
