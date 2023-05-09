---
title: Intro to Elasticsearch and Kibana笔记
date: 2022-10-15 16:40:56
categories: ["ELK"]
tags: ["command"]
---

# Intro to Elasticsearch and Kibana笔记

## 1. [Intro to Elasticsearch and Kibana](https://github.com/LisaHJung/Part-1-Intro-to-Elasticsearch-and-Kibana)

### 1.1 启动es

1. 通过`docker compose`，仓库里配置好了直接`docker compose up -d`即可

   ```bash
   git clone https://github.com/LisaHJung/Part-1-Intro-to-Elasticsearch-and-Kibana.git
   cd Part-1-Intro-to-Elasticsearch-and-Kibana
   docker compose up -d
   ```

   

### 1.2. 基本CRUD操作

```json
# cluster health
GET _cluster/health

# nodes info
GET _nodes/stats

# C create index
PUT favorite_candy

# C index doc without id
POST favorite_candy/_doc
{
  "first_name": "Lisa",
  "candy": "Sour Skittles"
}

# C index doc with id, delete+insert
PUT favorite_candy/_doc/1
{
  "first_name": "John",
  "candy": "Starburst"
}
PUT favorite_candy/_doc/2
{
  "first_name": "Rachel",
  "candy": "Rolos"
}
PUT favorite_candy/_doc/3
{
  "first_name": "Tom",
  "candy": "Sweet Tarts"
}

# C create doc with id, if conflict 409
PUT favorite_candy/_create/1
{
  "first_name": "Finn",
  "candy": "Jolly Ranchers"
}

# U update doc withid
POST favorite_candy/_update/1
{
  "doc": {
    "candy": "M&M's"
  }
}

# R read doc with id
GET favorite_candy/_doc/1

# R read all docs
GET favorite_candy/_search
{
  "query": {
    "match_all": {}
  }
}

# D delete doc with id
DELETE favorite_candy/_doc/1
```

### 1.3 课后练习

1. Create an index called `destinations`.

   ```json
   # req
   PUT destinations
   # res:
   {
     "acknowledged" : true,
     "shards_acknowledged" : true,
     "index" : "destinations"
   }
   ```

   

2. Pick five dream travel destinations. For each destination, index a document containing the name and the country. 

   ```json
   # _create id
   PUT destinations/_create/1
   {
     "name": "North Andros",
     "country": "The Bahamas"
   }
   PUT destinations/_create/2
   {
     "name": "Giza",
     "country": "Egypt"
   }
   PUT destinations/_create/3
   {
     "name": "Antofagasta de la Sierra Department",
     "country": "Argentina"
   }
   # _doc id
   PUT destinations/_doc/4
   {
     "name": "Herefordshire",
     "country": "United Kingdom"
   }
   # _doc without id
   POST destinations/_doc
   {
     "first_name": "Lisa",
     "candy": "Sour Skittles"
   }
   ```

   

3. Read(GET) each document to check the content of the document.

   ```json
   # req
   GET destinations/_doc/1
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "1",
     "_version" : 1,
     "_seq_no" : 0,
     "_primary_term" : 1,
     "found" : true,
     "_source" : {
       "name" : "North Andros",
       "country" : "The Bahamas"
     }
   }
   
   GET destinations/_doc/2
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "1",
     "_version" : 1,
     "_seq_no" : 0,
     "_primary_term" : 1,
     "found" : true,
     "_source" : {
       "name" : "North Andros",
       "country" : "The Bahamas"
     }
   }
   
   GET destinations/_doc/3
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "3",
     "_version" : 1,
     "_seq_no" : 2,
     "_primary_term" : 1,
     "found" : true,
     "_source" : {
       "name" : "Antofagasta de la Sierra Department",
       "country" : "Argentina"
     }
   }
   
   GET destinations/_doc/4
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "4",
     "_version" : 2,
     "_seq_no" : 5,
     "_primary_term" : 1,
     "found" : true,
     "_source" : {
       "name" : "Herefordshire",
       "country" : "United Kingdom"
     }
   }
   
   GET destinations/_doc/5CSL24MBvzmG0l5XVsoc
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "5CSL24MBvzmG0l5XVsoc",
     "_version" : 1,
     "result" : "created",
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "_seq_no" : 6,
     "_primary_term" : 1
   }
   ```

   

4. Update a field of a document.

   ```json
   POST destinations/_update/5CSL24MBvzmG0l5XVsoc
   {
     "doc": {
     "name": "Lisa",
     "country": "Sour Skittles"
     }
   }
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "5CSL24MBvzmG0l5XVsoc",
     "_version" : 2,
     "result" : "updated",
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "_seq_no" : 7,
     "_primary_term" : 1
   }
   ```

   

5. Read(GET) the updated document to ensure that the field has been updated.

   ```json
   GET destinations/_doc/5CSL24MBvzmG0l5XVsoc
   # res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "5CSL24MBvzmG0l5XVsoc",
     "_version" : 2,
     "_seq_no" : 7,
     "_primary_term" : 1,
     "found" : true,
     "_source" : {
       "name" : "Lisa",
       "country" : "Sour Skittles"
     }
   }
   ```

   

6. Delete a document of one place.

   ```json
   DELETE destinations/_doc/4ySC24MBvzmG0l5XZcoh
   
   #res
   {
     "_index" : "destinations",
     "_type" : "_doc",
     "_id" : "4ySC24MBvzmG0l5XZcoh",
     "_version" : 2,
     "result" : "deleted",
     "_shards" : {
       "total" : 2,
       "successful" : 1,
       "failed" : 0
     },
     "_seq_no" : 8,
     "_primary_term" : 1
   }
   ```

   

7. Copy and paste the following request to return all documents from the `destinations` index. 
   This is a great way to check whether all the CRUD operations you have performed thus far have worked!

```json
GET destinations/_search
{
  "query": {
    "match_all": {}
  }
}
# res
{
  "took" : 1312,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "destinations",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "North Andros",
          "country" : "The Bahamas"
        }
      },
      {
        "_index" : "destinations",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Giza",
          "country" : "Egypt"
        }
      },
      {
        "_index" : "destinations",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "Antofagasta de la Sierra Department",
          "country" : "Argentina"
        }
      },
      {
        "_index" : "destinations",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Herefordshire",
          "candy" : "United Kingdom"
        }
      },
      {
        "_index" : "destinations",
        "_type" : "_doc",
        "_id" : "4ySC24MBvzmG0l5XZcoh",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Lisa",
          "candy" : "Sour Skittles"
        }
      }
    ]
  }
}
```

