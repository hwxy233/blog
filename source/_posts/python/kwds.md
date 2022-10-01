---
title: Function argument **kwds
date: 2022-10-01 11:49:05
categories: ["Python"]
tags: ["Basic"]
---

# 关于函数里的**kwds

今天看`python`的`tutorial`[4.7节函数相关](https://docs.python.org/zh-cn/3.9/tutorial/controlflow.html#unpacking-argument-lists)发现有一个例子:

```python
def foo(name, **kwds):
    return 'name' in kwds
  
# 调用报错
foo(1, **{'name': 2})

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: foo() got multiple values for argument 'name'
```

我一开始以为，`**{'name': 2}`这个整体传给函数的是一个引用，但是后面看到了`**`是解包的意思，才理解，这里:

```python
**{'name': 2}
相当于name=2
```

所以执行会报错。
