---
layout: post
title: "Elastic Search"
author: "Sooyeon"
---
# [기본 개념](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-basic-concepts.html)

- NTR(Near Realtime): 문서를 인덱싱하는 시점부터 문서가 검색 가능해지기까지 약간의 대기 시간 존재(대개 1초)


- 클러스터: 하나 이상의 노드(서버)가 모인 것. 유니크한 이름으로 식별됨


- 노드: 클러스터에 포함된 단일 서버. 데이터를 저장하고 클러스터의 인덱싱 및 검색 기능에 참여. 클러스터처럼 이름으로 식별.


- 타입: 색인을 논리적으로 분류/구분한 것. 하나의 인덱스에서 하나 이상의 타입 정의 가능.
>ex) 블로그 플랫폼에서 모든 데이터를 하나의 인덱스에 저장한다면? 해당 인덱스에서 사용자, 블로그, 댓글 데이터에 대한 타입을 각각 정의 가능.


- 도큐먼트: 기본 정보 단위. 도큐먼트가 모여서 인덱스가 된다.


- 샤드: 인덱스는 방대한 양의 데이터 저장 가능. 이 데이터가 단일 노드의 하드웨어 한도를 초과하는 경우, 샤드라는 조각으로 분할할 수 있음. 인덱스 생성 시 샤드 수 정의 가능.


- 레플리카: 인덱스의 샤드에 대해 하나 이상의 복사본을 생성한 것.


> 인덱스는 여러 개의 샤드로 분할 가능. 하나의 인덱스는 복제하지 않거나 1회 이상 복제 가능(기본 샤드 + 레플리카 샤드).

> 인덱스 생성 후 레플리카 수는 변경 가능하나, 샤드 수는 변경 불가하다.

> 기본적으로 es의 각 인덱스는 기본 샤드 5개, 레플리카 1개를 가짐.

# Mac에서 Elastic Search 시작하기

es 다운로드
```
% brew install elasticsearch
```

es 실행
```
% elasticsearch
```

es 실행(클러스터 및 노드 이름 재정의)
```
elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

# 클러스터 둘러보기
## [클러스터 상태](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-cluster-health.html)
_cat API를 사용한다.
아래의 명령어 실행
```
GET /_cat/health?v
```
curl로 요청하려면
```
curl -X GET "localhost:9200/_cat/health?v"
```

다음과 같이 응답함.
```
epoch      timestamp cluster         status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1625486032 11:53:52  my_cluster_name green           1         1      0   0    0    0        0             0                  -                100.0%
```
status
- green: 모두 양호한 상태(클러스터 정상 작동중)
- yello: 모든 데이터가 사용 가능한 상태이지만 일부 레플리카 아직 배정되지 않은 상태(클러스터 정상 작동중)
- red: 일부 데이터 사용할 수 없는 상태(클러스터 부분적으로 작동중 -> 사용 가능 샤드에서 계속 검색 요청을 처리)

노드 목록을 보려면
```
GET /_cat/nodes?v
```

## [모든 인덱스 나열](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-list-all-indices.html)
```
GET /_cat/indices?v
```

## [인덱스 생성](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-create-index.html)
customer라는 인덱스 만들고, 모든 인덱스 나열
```
PUT /customer?pretty
GET /_cat/indices?v
```
다음과 같은 응답이 옴
```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer kHCNmiRqTv2MvGuF-3R4qA   1   1          0            0       208b           208b
```
customer 인덱스가 yellow인 이유는?
es가 이 인덱스가 레플리카를 1개 배정했기 때문.
그러나 노드가 하나밖에 없어서 아직 레플리카가 배정되지 않았다.