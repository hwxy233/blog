---
title: Change Hexo Theme
date: 2022-08-28 18:50:58
categories: ["Blog"]
tags: ["Hexo","Hexo Theme"]
---

# Hexo修改主题遇到的问题

## 1. 部署时报错

1. 使用了`cactus`这款主题在本地运行没有问题

2. 但是`git push`推送时出现了报错

```bash
6:10:34 PM: Error checking out submodules: fatal: No url found for submodule path 'themes/cactus' in .gitmodules
6:10:34 PM: Creating deploy upload records
6:10:34 PM: Failing build: Failed to prepare repo
6:10:34 PM: Failed during stage 'preparing repo': Error checking out submodules: fatal: No url found for submodule path 'themes/cactus' in .gitmodules
: exit status 128
6:10:34 PM: Finished processing build request in 3.922570239s
```

3. 原因是设置主题时直接`git clone`到了`theme`目录下，没有添加`git submodule`

## 2. 解决办法

1. 先备份下主题的配置文件`_config.yml`

```bash
cp _config.yml ../cactus_config.yml
```

2. 删除主题再重新添加

```bash
git rm -r themes/cactus --cached

rm -rf themes/cactu
```

3. 重新添加`submodule`

```bash
git submodule add https://github.com/probberechts/hexo-theme-cactus.git themes/cactus
```

生成了`.gitmodule`

```yml
[submodule "themes/cactus"]
        path = themes/cactus
        url = https://github.com/probberechts/hexo-theme-cactus.git
```

4. 把刚才备份的主题配置拷贝回去

```bash
cd themes

cp cactus_config.yml cactus/_config.yml
```

5. 提交和重新部署

```bash
git add .

git commit -m "Bug Fix: theme set."

git push
```

6. 成功

```bash
7:34:33 PM: Started saving maven dependencies
7:34:33 PM: Finished saving maven dependencies
7:34:33 PM: Started saving boot dependencies
7:34:33 PM: Finished saving boot dependencies
7:34:33 PM: Started saving rust rustup cache
7:34:33 PM: Finished saving rust rustup cache
7:34:33 PM: Started saving go dependencies
7:34:33 PM: Finished saving go dependencies
7:34:33 PM: Build script success
7:34:33 PM: Uploading Cache of size 121.2MB
7:34:34 PM: Finished processing build request in 17.453636981s
7:34:36 PM: Site is live ✨
```

