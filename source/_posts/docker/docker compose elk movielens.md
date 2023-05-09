---
title: 使用docker-compose启动elk并导入movielens数据集
date: 2022-10-15 14:41:00
categories: ["Docker"]
tags: ["docker-compose"]
---

# 使用docker-compose启动elk并导入movielens数据集

初学`es`打算用`docker-compose`进行启动，想导入`movielens`数据但是遇到了一些问题。

主要是`logstash`的问题，以及`xpack`角色权限的问题，这里记录一下`bugfix`后的启动步骤。

## 1. docker-compose地址

经过搜索，有写好的开源项目，非常不错:

```bash
git clone https://github.com/deviantony/docker-elk.git
```

## 2. 步骤

分支基于`release-7.x`，新建一个`release-7.x-test1`

### 2.1 修改xpacck

1. 修改`elasticsearch/config/elasticsearch.yml`里的`xpack`配置，这里保留了认证功能

```
xpack.license.self_generated.type: basic
```

### 2.2 下载moveielens数据集

```bash
mkdir logstash/movielens
cd logstash/movielens
curl -O https://files.grouplens.org/datasets/movielens/ml-latest-small.zip
unzip ml-latest-small.zip
```

### 2.3 增加一个logstash的pipeline

```bash
cd logstash/pipeline
vi movielens.conf
```

1. `input`里的`path`是刚才下载的数据集的位置，待会在`docker-compose`里定义映射
2. `output`里的`hosts`，`user`，`password`分别代表这个集群里`es`的地址，`logstash`写入`es`的用户名和密码，这个用户在`.env`中进行了定义

```json
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  file {
    path => "/usr/share/logstash/movielens/ml-latest-small/movies.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
  csv {
    separator => ","
    columns => ["id", "content", "genre"]
  }
  
  mutate {
    split => { "genre" => "|"}
    remove_field => ["path", "host", "@timestamp", "message"]
  }
  
  mutate {
    split => { "content" => "(" }
    add_field => { "title" => "%{[content][0]}"}
    add_field => { "year" => "%{[content][1]}"}
  }
  
  mutate {
    convert => {
      "year" => "integer"
    }
    strip => ["title"]
    remove_field => ["path", "host", "@timestamp", "content"]
  }
}

output {
  elasticsearch {
    hosts => "elasticsearch:9200"
    index => "movies"
    document_id => "%{id}"
    user => "logstash_internal"
    password => "${LOGSTASH_INTERNAL_PASSWORD}"
  }
  stdout {}
}

```

3. 增加一个`pipeline.conf`

```bash
cd logstash/config
vi pipelines.yml
```

`path.config`指向了`pipeline`会在`logstash`启动时执行

```yaml
pipeline.id: movielens
path.config: "/usr/share/logstash/pipeline/movielens.conf"
pipeline.id: tcp
path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

### 2.4 修改docker-compose.yml

1. 修改`logstash`的配置

在`volumes`里增加`pipelines.yml`的映射，以及`movielens`数据集的映射

```yaml
  logstash:
    build:
      context: logstash/
      args:
        ELASTIC_VERSION: ${ELASTIC_VERSION}
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro,Z
      - ./logstash/config/pipelines.yml:/usr/share/logstash/config/pipelines.yml:ro,Z
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro,Z
      - ./logstash/movielens:/usr/share/logstash/movielens:ro,Z
    ports:
      - "5044:5044"
      - "50000:50000/tcp"
      - "50000:50000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: -Xms256m -Xmx256m
      LOGSTASH_INTERNAL_PASSWORD: ${LOGSTASH_INTERNAL_PASSWORD:-}
    networks:
      - elk
    depends_on:
      - elasticsearch
```

### 2.5 修改logstash的角色权限

1. 对`logstash_writer`这个角色的索引权限增加`movies`的权限，对应`pipeline`的`output`的索引

```bash
vi setup/roles/logstash_writer.json
```

```json
         "write",
         "manage"
       ]
     },
     {
       "names": [
         "movies"
       ],
       "privileges": [
         "all"
       ]
     }
   ]
 }
```

## 3. 启动

```bash
# build
docker compose build
# start
docker compose up -d

# stop 不想要volumns数据的话可以 -v
docker down -v
```

