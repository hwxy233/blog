---
title: Hexo修改主题遇到的问题
date: 2022-08-28 18:50:58
categories: ["Hexo"]
tags: ["theme"]
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

# 3. 这样虽然成功了可是自己的`_config.yml`没有生效

1. 因为`submodule`是对目标项目某次提交的索引
2. 而对配置文件的修改在提交后如果不`push`到目标项目的话部署时是拉取不到对应的提交的

```bash
8:11:15 PM: Creating deploy upload records
8:11:15 PM: Failing build: Failed to prepare repo
8:11:15 PM: Failed during stage 'preparing repo': Error checking out submodules: Submodule 'themes/cactus' (https://github.com/probberechts/hexo-theme-cactus.git) registered for path 'themes/cactus'
Cloning into '/opt/build/repo/themes/cactus'...
fatal: remote error: upload-pack: not our ref 5cfea976a4076cb4b51f0aba79ac7aa9f27b1cee
fatal: Fetched in submodule path 'themes/cactus', but it did not contain 5cfea976a4076cb4b51f0aba79ac7aa9f27b1cee. Direct fetching of that commit failed.
: exit status 128
8:11:15 PM: Finished processing build request in 2.584416391s
```

# 4. fork主题的代码

1. 既然这样只能`fork`下主题的仓库
2. 然后添加`submodule`时用自己`fork`的仓库地址，这样就可以使用自己的配置文件了
3. 删除`submodule`

```bash
rm -rf themes/cactus

rm -rf /.git/modules/*

# 然后删除.gitmodules里面的内容

# 然后再删除.git/config里面的内容
```

4. 修改`fork`的项目然后提交
5. 重新添加`submodule`

```bash
```

