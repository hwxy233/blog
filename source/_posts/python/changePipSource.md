---
title: Change pip source
date: 2022-09-02 21:17:26
categories: ["Python"]
tags: ["Env"]
---

# Mac系统更换pip源

想给`pip`换清华源，偶然看到了环境变量的一些情况。

## 1. 查看环境变量

今天使用`Pycharm`时自动识别到了`python`解释器，路径为:

```bash
/usr/local/bin/python3
/usr/local/bin/python3.10
/usr/bin/python3
```

这里发现有3个解释器，另外我想起来并没有配置过`python`的环境变量，安装是通过`hombrew`安装的: 

```bash
❯ brew info python3
==> python@3.10: stable 3.10.6 (bottled)
Interpreted, interactive, object-oriented programming language
https://www.python.org/
/usr/local/Cellar/python@3.10/3.10.6_1 (3,130 files, 56.8MB) *
  Poured from bottle on 2022-08-19 at 21:47:54
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/python@3.10.rb
License: Python-2.0
==> Dependencies
Build: pkg-config ✘
Required: gdbm ✔, mpdecimal ✔, openssl@1.1 ✔, readline ✔, sqlite ✔, xz ✔
==> Caveats
Python has been installed as
  /usr/local/bin/python3

Unversioned symlinks `python`, `python-config`, `pip` etc. pointing to
`python3`, `python3-config`, `pip3` etc., respectively, have been installed into
  /usr/local/opt/python@3.10/libexec/bin
```

这里看到创建了软连接:

```bash
❯ pwd
/usr/local/bin
❯ ls -li | grep python3
42516027 lrwxr-xr-x  1 hwxy  admin   42  8 19 21:48 python3 -> ../Cellar/python@3.10/3.10.6_1/bin/python3
42516028 lrwxr-xr-x  1 hwxy  admin   49  8 19 21:48 python3-config -> ../Cellar/python@3.10/3.10.6_1/bin/python3-config
42516029 lrwxr-xr-x  1 hwxy  admin   45  8 19 21:48 python3.10 -> ../Cellar/python@3.10/3.10.6_1/bin/python3.10
42516030 lrwxr-xr-x  1 hwxy  admin   52  8 19 21:48 python3.10-config -> ../Cellar/python@3.10/3.10.6_1/bin/python3.10-config
```

但是要注意`/usr/bin/python3`这个是系统自己安装的:

```bash
❯ pwd
/usr/bin
❯ ll | grep python3
-rwxr-xr-x  76 root   wheel   163K  8 11 14:44 python3
❯ ./python3
Python 3.8.2 (default, Apr  8 2021, 23:19:18)
[Clang 12.0.5 (clang-1205.0.22.9)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```



## 2. 关于$PATH

```bash
❯ echo $PATH
/Users/hwxy/.tiup/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Users/hwxy/codes/go:/usr/local/go/bin:./node_modules/.bin
```

可以看到会自动的添加`/usr/local/bin`下的命令道环境变量，这样确实不用添加环境变量了。

这些`xxx/bin`的区别简单来说就是系统的和自己装的，以及普通用户的和超级管理员的区别，具体参见:

[differences between /bin /sbin /usr/bin /usr/sbin /usr/local/bin usr/local](https://askubuntu.com/questions/308045/differences-between-bin-sbin-usr-bin-usr-sbin-usr-local-bin-usr-local)



## 2. 设置pip源

```bash
# /usr/local/bin下的是python3而不是python
python3 -m pip install --upgrade pip

# homebrew info显示的是pip3
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

 
