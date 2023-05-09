---
title: Import该怎么用
date: 2022-09-11 21:15:28
categories: ["Python"]
tags: ["language"]
---

# Import该怎么用

由于初学`python`，有时候还是会按照`java`的方式去写，比如会把一些公共的方法放到同一个文件，然后需要的时候直接使用这个文件的方法，这时就会用到`import`，但是有时候会遇到一些报错(尤其是相对路径import的时候)。

总之`__main__`放到最外层执行的话，无论怎么引用就都可以了XD。

## 1. `sys.path`和`__name__`

为什么先说这2个东西呢，因为这涉及到了绝对路径引用和相对路径引用的一些执行规则。

### 1.1 `sys.path`

当想要`import`一个文件的方法时，往往会这样写:

```python
import xxx
或者是
from xxx import xxx
```

这就涉及到一个问题，就是`python`解释器是怎么知道这个`imnport`的xxx在哪里呢？答案是从`sys.path`里面去找，找不到就会报错。

打印一下`sys.path`看看:

```python
# module/run_path.py
import sys

print(sys.path)
```

在终端执行的结果:

```bash
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module', '/Users/hwxy/opt/anaconda3/lib/python39.zip', '/Users/hwxy/opt/anaconda3/lib/python3.9', '/Users/hwxy/opt/anaconda3/lib/python3.9/lib-dynload', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages/aeosa']
 
```

注意`/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module`这个路径，这是执行的脚本所在的路径，在脚本执行时会自动添加到`sys.path`中。

当`import`执行时会从这些目录里面去找被引用的文件是否存在。待会用一些测试代码来验证。

### 1.2 `__name__`

`sys.path`是绝对路径引用的搜索依据，那么`__name__`则是间接引用的搜索依据。

来打印下看看:

```python
print(__name__)
```

直接在脚本内打印:

```bash
__main__
```

由其他脚本调用打印:

```bash
abs_import.a1
```

经常会看到这样的代码块:

```python
if __name__ == "__main__":
    xxx
```

这是为了防止被调用时，方法外的测试代码执行，因为在脚本内执行，`__name__`是`__main__`可以将测试的代码放到这里面，这样被调用时`__name__`就不是`__main__`了测试代码就不会执行了。



## 2. 测试绝对引用

以这个`module`目录为基础，这里面有2个子目录`abs_import`和`rel_import`，做一些测试。

```bash
❯ pwd
/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module
❯ tree -L 1
.
├── __pycache__
├── abs_import
├── rel_import
└── run_path.py

3 directories, 1 file

❯ cd abs_import
❯ pwd
/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import
❯ tree -L 1
.
├── __pycache__
└── a1.py

1 directory, 1 file

```

### 2.1 父目录引用子目录

`a1.py`:

```python
def f1():
    print("f1 in a1")
```

在`run_path.py`引用`a1.py`:

```python
import a1

a1.f1()

# 这样写会报错:
Traceback (most recent call last):
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/run_path.py", line 1, in <module>
    import a1
ModuleNotFoundError: No module named 'a1' s s

```

打印下`sys.path`看看:

```python
import sys

print(sys.path)

# 输出,可以看到有运行的当前目录
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module', ...]
```

由于`a1`在`abs_import`下面，直接从`module`下面找肯定是没有的，所以改成这样:

```python
import abs_import.a1 as a1

a1.f1()

# 输出
f1 in a1
```

另一种办法:

```python
import os
import sys
# 手动将子目录append到path中,这样就能直接搜索到a1了
sys.path.append(os.path.abspath(os.path.join(os.getcwd(), "abs_import")))

print(sys.path)

import a1

a1.f1()

# 输出
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module', ..., '/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import']
f1 in a1
```

### 2.2 子目录引用子目录

和`a1`在同级别目录创建一个`a2.py`:

```python
def f2():
    print("f2 im a2")
```

以及一个`run_a1_a2.py`作为运行的脚本文件:

```bash
❯ tree -L 1
.
├── __pycache__
├── a1.py
├── a2.py
└── run_a1_a2.py

1 directory, 3 files

```

在`run_a1_a2`中引用`a1`和`a2`:

```python
# 因为是同级目录所以可以直接import
import a1
import a2
import sys

print(sys.path)
a1.f1()
a2.f2()

# 输出
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import', '/Users/hwxy/opt/anaconda3/lib/python39.zip', '/Users/hwxy/opt/anaconda3/lib/python3.9', '/Users/hwxy/opt/anaconda3/lib/python3.9/lib-dynload', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages/aeosa']
f1 in a1
f2 im a2

```

### 2.3 子目录引用父目录

因为执行脚本所在的目录会被添加到`path`中，所以执行的脚本在父目录和子目录被添加到`sys.path`的路径不同。

新创建一个`a3`:

```python
def f3():
    print("f3 in a3")

```

```bash
❯ pwd
/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module

❯ tree -L 1
.
├── __pycache__
├── a3.py
├── abs_import
├── rel_import
└── run_path.py

3 directories, 2 files

```

#### 2.3.1 执行的脚本在子目录

修改`run_a1_a2`:

```python
import sys

print(sys.path)
import a3

a3.f3()

# 输出，会报错
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import', '/Users/hwxy/opt/anaconda3/lib/python39.zip', '/Users/hwxy/opt/anaconda3/lib/python3.9', '/Users/hwxy/opt/anaconda3/lib/python3.9/lib-dynload', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages', '/Users/hwxy/opt/anaconda3/lib/python3.9/site-packages/aeosa']
Traceback (most recent call last):
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import/run_a1_a2.py", line 9, in <module>
    import a3
ModuleNotFoundError: No module named 'a3'
```

因为`sys.path`只是将当前执行脚本的路径加到`sys.path`中，没有父目录，也就找不到父目录里的`a3`。

解决办法自然是将父目录加到`path`中:

```python
import os
import sys

sys.path.append(os.path.abspath(os.path.join(os.getcwd(), "..")))
import a3

print(sys.path)
a3.f3()

# 输出
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/abs_import', ..., '/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module']
f3 in a3

```

#### 2.3.2 执行的脚本在父目录

上面那样做还是很麻烦原因在于执行文件在子目录，但是如果在父目录就不需要了。

修改`run_path`:

```python
import abs_import.run_a1_a2 as run_a1_a2
import sys

print(sys.path)
run_a1_a2.test_a3()
```

修改`run_a1_a2`:

```python
import a3


def test_a3():
    a3.f3()
```

运行`run_path`:

```bash
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module', ...]
f3 in a3
```

因为`a3`在父目录下，所以直接找可以找到。



## 3. 测试相对引用

### 3.1 相对目录表示

```bash
# 当前目录
.
## 父目录
..
## 父目录的父目录
...
```

在`rel_import`下新建`a4`，`a5`。

```bash
❯ cd rel_import
❯ tree -L 1
.
├── __pycache__
├── a4.py
└── a5.py

1 directory, 2 files


```



### 3.2 引用同级目录

在`a5`中使用相对引用`a4`:

```python
# a4.py

def f4():
    print("f4 im a4")

```

```python
# a5.py
from .a4 import f4


def f5():
    print("f5 im a5")


f4()

# 运行会报错
Traceback (most recent call last):
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/rel_import/a5.py", line 1, in <module>
    from .a4 import f4
ImportError: attempted relative import with no known parent package
```

但是如果在父目录引用`a5`运行呢？

```bash
❯ tree -L 1
.
├── __pycache__
├── a3.py
├── abs_import
├── rel_import
├── run_a4_a5.py
└── run_path.py

3 directories, 3 files

```

新建一个`run_a4_a5`:

```python
import rel_import.a5 as a5

a5.f5()

# 输出
f4 im a4
f5 im a5
```

发现没有问题，打印下a5和a4的`__name__`看看:

```bash
a4__name__: rel_import.a4
a5__name__: rel_import.a5
f4 im a4
f5 im a5
```

那么尝试给`a5`改下`__name__`试试:

```python
# a5.py
__name__ = "rel_import.a5"

print("a5__name__:", __name__)
from .a4 import f4


def f5():
    print("f5 im a5")


f4()

# 输出
a5__name__: rel_import.a5
Traceback (most recent call last):
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/rel_import/a5.py", line 4, in <module>
    from .a4 import f4
ModuleNotFoundError: No module named 'rel_import'
```

和刚才的报错不同了，这次找不到了`rel_import`原因很简单，因为`sys.path`里没有`rel_import`，我们尝试加下试试:

```python
# a5.py
__name__ = "rel_import.a5"

import os.path

print("a5__name__:", __name__)

import sys

sys.path.append(os.path.abspath(os.path.join(os.getcwd(), "..")))
print(sys.path)

from .a4 import f4


def f5():
    print("f5 im a5")


f4()

# 输出
a5__name__: rel_import.a5
['/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/rel_import', ..., '/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module']
a4__name__: rel_import.a4
f4 im a4
```

这回可以了，好麻烦啊。。。也就是说:

1. 要保证父目录要在`sys.path`下才可以
2. 或者执行的脚本在父目录，子目录下的脚本怎么引用就无所谓了

### 3.3 引用父目录

在`rel_import`新建目录`rel_sub`

```bash
❯ cd rel_import/rel_sub
❯ tree -L 1
.
└── a6.py

0 directories, 1 file

```

修改`a6`来引用a4:

```python
from ..a4 import f4


def f6():
    f4()
    print("f6 im a5")

```

修改`run_a4_a5`:

```python
import rel_import.rel_sub.a6 as a6

a6.f6()

# 输出
a4__name__: rel_import.a4
f4 im a4
f6 im a5
```

### 3.4 引用父目录的父目录

修改`a6`:

```python
# a6.py
print("a6__name__:", __name__)

from ..a4 import f4
from ...a3 import f3


def f6():
    f3()
    f4()
    print("f6 im a5")

```

执行`run_a4_a5`:

```bash
a6__name__: rel_import.rel_sub.a6
a4__name__: rel_import.a4
Traceback (most recent call last):
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/run_a4_a5.py", line 9, in <module>
    import rel_import.rel_sub.a6 as a6
  File "/Users/hwxy/codes/pythonProjects/numpyAndPandasPlay/basic/module/rel_import/rel_sub/a6.py", line 4, in <module>
    from ...a3 import f3
ImportError: attempted relative import beyond top-level package
```

意思是找不到`...`这个第三个`.`的层级，`.`的个数要和`__name__`里`.`的个数匹配上，也就是说引用不到最外层的脚本文件。但是可以用绝对引用的方式来进行。

```python
# a6.py
from ..a4 import f4
from a3 import f3


def f6():
    f3()
    f4()
    print("f6 im a5")

    
# 输出
f3 in a3
f4 im a4
f6 im a5
```

