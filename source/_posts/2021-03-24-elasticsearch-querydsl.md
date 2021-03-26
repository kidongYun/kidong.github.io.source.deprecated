---
layout: post
title:  "ElasticSearch QueryDSL"
date:   2021-03-24 00:0054 +0900
categories: java
---

### 클러스터 정보 조회하기.

REQ
```
curl -XGET http://192.168.28.176:9200/_cluster/health?pretty
```

RES
```json
{
  "cluster_name" : "interCluster",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 348,
  "active_shards" : 348,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 562,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 38.24175824175824
}
```

yellow 상태는 모든 데이터의 읽기/쓰기가 가능한 상태이지만 일부 replica shard가 아직 배정되지 않은 상태를 말합니다.

### RAID 0 (STRIPE)
2개의 디스크가 있을때 데이터를 이 두개의 디스크에 나눠어 저장하는 방법. 디스크의 액세스 활동이 두 군데서 발생하기 때문에 데이터를 저장하는 속도를 높일 수 있다.

* 저장이 포인트. 검색이 아니다.

### RAID 1 (MIRROR)
2개의 디스크가 있을 때 각 디스크에 데이터를 똑같이 저장하는 방법. 성능을 높이지는 않지만 백업의 효과를 가진다.

### SHARD
RAID 0 와 유사한 개념. Shard 수치가 5라고 한다면, 하나의 인덱스(DB)에 내부적으로 논리적으로 5개의 샤드를 생성한다. 물론 또다른 index를 생성하게 되면 그에 대한 shard를 새로 생성된다.

### REPLICA
RAID 1 과 유사한 개념. 복제본을 가지고 있는 것으로. 원본 shard에 장애가 나더라 도 복제본 Shard를 통해 데이터를 제공할 수 있다.

한 노드안에는 PRIMARY, REPLICA SHARD가 서로 다르게 들어가게 되는데. 이는 특정 node가 장애가 났을 때 동일한 데이터를 잃지 않게 하기 위함이다.

또 REPLICA의 경우 동일한 데이터를 여러 SHARD에서 저장하고 있기 때문에 검색시에 성능향상을 가지고 있다.

* 검색이 포인트. 저장이 아니다.


### INDEX 생성하기.

```
curl -XPUT http://192.168.28.176:9200/test_index_by_ykd?pretty
```

### INDEX 조회하기.

```
curl -XGET http://192.168.28.176:9200/test_index_by_ykd?pretty
```

### INDEX 제거하기.

```
curl -XDELETE http://192.168.28.176:9200/test_index_by_ykd?pretty
```

### 현재 존재하는 인덱스 정보 조회하기.
REQ
```
curl -XGET http://192.168.28.176:9200/_cat/indices?v
```

RES
```
health status index                       pri rep docs.count docs.deleted store.size pri.store.size
yellow open   test_index_by_ykd             5   1          0            0       795b           795b
```

health 상태랑, primary shard 개수와, replica shard 개수를 알 수 있다.

### 현재 존재하는 샤드 정보 조회하기.

REQ
```
curl -XGET http://192.168.28.176:9200/_cat/shards?v
```

RES
```
index                       shard prirep state        docs   store ip             node
test_index_by_ykd           1     p      STARTED         0    159b 192.168.28.176 node-176-client
test_index_by_ykd           1     r      UNASSIGNED
test_index_by_ykd           3     p      STARTED         0    159b 192.168.28.176 node-176-client
test_index_by_ykd           3     r      UNASSIGNED
test_index_by_ykd           4     p      STARTED         0    159b 192.168.28.176 node-176-client
test_index_by_ykd           4     r      UNASSIGNED
test_index_by_ykd           2     p      STARTED         0    159b 192.168.28.176 node-176-client
test_index_by_ykd           2     r      UNASSIGNED
test_index_by_ykd           0     p      STARTED         0    159b 192.168.28.176 node-176-client
test_index_by_ykd           0     r      UNASSIGNED
```

shard 항목이 shard 번호를 보여주는 것 같고 indces 조회를 통해서 shard 개수가 5개인 걸로 볼 수 있는데 보면 shard 번호가.
1~5까지 있다. 2개씩있는 이유는 replica shard도 한개씩 더 있어야하기 때문이다.

replica shard 들이 모두 UNASSIGNED 상태에 있는데. 그 이유는 primary shard와 replica shard의 같은 번호들은 같은 노드에 존재할 수 없다. 백업의 의미가 없어지기 때문.




### 문자열을 직접 넣어 DOCUMENT 생성하기.

```
hello it's me
```