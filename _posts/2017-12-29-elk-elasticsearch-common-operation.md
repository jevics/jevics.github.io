---
layout: post
categories: ELK
title: Elasticsearch常用操作(一)
date: 2017-07-21 09:26:45 +0800
description: elk
keywords: elk elasticsearch
---


# sql接口查询

``` bash
sql 2.x
curl http://es-node/_sql -d "SELECT count(*) FROM index-test/type where timestamp=1484048580"

sql 5.x
http://es-node/_sql?sql=SELECT count(*) FROM index-test where @timestamp>=149925060000 and @timestamp<1499254200000

```


# 冷热分片

```
curl -XPUT $URL/${Inx}-${date}/_settings -d '{ "index.routing.allocation.include.zone": "hot"}'

curl -XPUT $URL/${Inx}-${date_ops}/_settings -d '{ "index.routing.allocation.include.zone": "stale"}'

```

# 索引优化分段

```
2.x 
curl -XPOST node-es:9200/teset-2017.07.05/_optimize?max_num_segments=1&wait_for_merge=false

5.x
curl -XPOST node-es:9200/test-2017.07.05/_forcemerge?max_num_segments=1
```
# 搜索
``` 
GET index/doc/_search
GET index/doc/_search?pretty
curl node-es:9200/index/doc/_search |python -mjson.tool
```
# 删除
- [官方API文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/docs-delete-by-query.html#docs-delete-by-query)

```
推荐使用: curl -XPOST _delete_by_query?scroll_size=2000&wait_for_completion=false

curl -XPOST node-es:9200/test-index/_delete_by_query -d '
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}'

2.x 
curl -XDELETE node-es:9200/test-index/_query -d '{
"query": { 
    "match": {
      "message": "some message"
    }
  }
}'

查询删除任务:
5.x
GET _tasks?detailed=true&actions=*/delete/byquery
2.x
GET _tasks?detailed=true&actions=*/delete/by_query
```

# 任务查询 _tasks
[官网API文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)

- es 2.x

```
GET /tasks
GET _cluster/pending_tasks
GET /_tasks?actions=*/list*
GET /_tasks?actions=*reindex,*by_query
返回的结果:
{
.....
      "tasks": {
        "w8S8uYR9ThelFe7u5blX-Q:466799605": {
          "node": "w8S8uYR9ThelFe7u5blX-Q",
          "id": 466799605,
          "type": "transport",
          "action": "indices:data/write/delete/by_query",
          "start_time_in_millis": 1499846410168,
          "running_time_in_nanos": 527881720186
.....
}


```
- es 5.x 

```
GET _cat/tasks?v&time
GET _tasks?detailed=true&actions=*/delete/byquery
GET _tasks?pretty&detailed&actions=*reindex,*byquery
返回结果:
{
.....
      "tasks": {
        "RiuComp3TOWhz0y804Ym7A:55597577": {
          "node": "RiuComp3TOWhz0y804Ym7A",
          "id": 55597577,
          "type": "transport",
          "action": "indices:data/write/delete/byquery",
          "status": {
            "total": 47683364,
            "updated": 0,
            "created": 0,
            "deleted": 15294000,
            "batches": 7648,
            "version_conflicts": 0,
            "noops": 0,
            .....
      }
    }
  }
}

取消任务:
POST _tasks/RiuComp3TOWhz0y804Ym7A:55597577/_cancel


```



# 添加模板
```
PUT  /_template/test
{
  
  "template":"test-*",
  "settings" : {
    "index.refresh_interval" : "60s"
  }
}
```
# 查询模板
```
GET _template/yflive

curl es-3:9200/bank/_count?pretty -d '
{ 
"query": {
   "match" : { "city" : "SZ" }
}
}'

```

# 调整查询窗口大小

```
get index-test/_settings

PUT index-test/_settings
{ 
  "index" : { 
    "max_result_window" : 500000 
  } 
}
```

# 磁盘配额
```
PUT _cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.disk.watermark.low" : "90%",
        "cluster.routing.allocation.disk.watermark.high" : "10gb"
    }
}

```

# 更新文档数据

```
POST nginx-log/logs/AVxiwZcu_Z9aU351KXOL/_update
{
  "doc": {
    "value" : 732439
  }
}

```

# 指定索引分片分配到某个特定主机
[官网文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-allocation-filtering.html)

```
PUT test/_settings
{
  "index.routing.allocation.include._ip": "192.168.2.*"
}

```

- [Index Modules](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html)

转载请注明出处，本文采用 [CC4.0](http://creativecommons.org/licenses/by-nc-nd/4.0/) 协议授权
