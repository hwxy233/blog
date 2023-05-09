---
title: Run Docker without root on Ubuntu
date: 2023-05-09 21:42:28
categories: ["Docker"]
tags: ["env"]

---

# 在Ubuntu中使用非root用户运行Docker

## 1. 将要运行Docker的用户添加到docker组中

```bash
sudo usermod -aG docker 要运行Docker的用户

eg:
sudo usermod -aG docker hwxy
```

## 2. 重启Docker

```bash
# Ubuntu中使用service重启
service docker restart
```

重启之后需要打开一个新的终端生效