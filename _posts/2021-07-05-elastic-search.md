---
layout: post
title: "Elastic Search"
author: "Sooyeon"
---
# [기본 개념](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-basic-concepts.html)

- NTR(Near Realtime): 문서를 인덱싱하는 시점부터 문서가 검색 가능해지기까지 약간의 대기 시간 존재(대개 1초)


- 클러스터: 하나 이상의 노드(서버)가 모인 것. 유니크한 이름으로 식별됨


- 노드: 클러스터에 포함된 단일 서버. 데이터를 저장하고 클러스터의 인덱싱 및 검색 기능에 참여. 클러스터처럼 이름으로 식별.


- 인덱스: 비슷한 특성을 가진 도큐먼트의 모음. 인덱스는 이름으로 식별됨. 단일 클러스터에서 원하는 개수의 색인 정의 가능. 


- 타입: 인덱스를 논리적으로 분류/구분한 것. 하나의 인덱스에서 하나 이상의 타입 정의 가능.
>ex) 블로그 플랫폼에서 모든 데이터를 하나의 인덱스에 저장한다면? 해당 인덱스에서 사용자, 블로그, 댓글 데이터에 대한 타입을 각각 정의 가능.


- 도큐먼트: 기본 정보 단위. 도큐먼트가 모여서 인덱스가 된다.


- 샤드: 인덱스는 방대한 양의 데이터 저장 가능. 이 데이터가 단일 노드의 하드웨어 한도를 초과하는 경우, 샤드라는 조각으로 분할할 수 있음. 인덱스 생성 시 샤드 수 정의 가능.
    - 샤딩이 필요한 이유
        - 콘텐츠 볼륨의 수평 분할/확장이 가능해짐
        - 작업을 여러 샤드에 분산 배치하고 병렬화함으로써 성능/처리량 늘릴 수 있음
    

- 레플리카: 인덱스의 샤드에 대해 하나 이상의 복사본을 생성한 것.
    - replication이 필요한 이유
        - 샤드/노드 오류가 발생하더라도 고가용성 제공. (기본 샤드와 동일한 노드에 배정되지 않는다)
        - 모든 레플리카에서 병렬 방식으로 검색 실행 => 검색 볼륨/처리량 확장 가능.


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

## [도큐먼트 인덱싱 및 쿼리](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-index-query.html)
customer 인덱스에 도큐먼트 추가
도큐먼트를 인덱싱하려면 es에 인덱스의 어떤 유형을 선택할 지 알려줘야 한다.


다음과 같은 도큐먼트를 만들어서 customer 인덱스에 external 유형으로 인덱싱한다. (ID=1)
```
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
```
성공적으로 도큐먼스 생성되면 다음과 같은 응답이 옴
만약 customer 인덱스가 생성되지 않았던 상태라면 자동으로 인덱스가 생성됨.
```
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
```
인덱스 검색
```
GET /customer/external/1?pretty
```
응답
```
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

## [인덱스 삭제](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-delete-index.html)
customer 인덱스 삭제
```
DELETE /customer?pretty
GET /_cat/indices?v
```
응답
```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```
인덱스가 삭제되어 클러스터에 아무 것도 없는 상태가 됨.

es에서 데이터에 엑세스 하는 방식의 패턴
```
<REST Verb>/<Index>/<Type>/<ID>
```

# [데이터 수정](https://www.elastic.co/guide/kr/elasticsearch/reference/current/gs-modifying-data.html)
es는 실시간에 가깝게 데이터 조작, 검색 기능을 제공함.
데이터 인덱싱/업데이터/삭제 시점부터 검색 결과에 나타나기까지 1초 정도 걸림(새로고침 간격).
트랜잭션이 완료되면 즉시 데이터 사용 가능해지는 다른 DB와 구별되는 특징.

## 문서 인덱싱/대체
```
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}

PUT /customer/external/1?pretty
{
  "name": "Jane Doe"
}
```
위와 같이 인덱스가 존재하는 상태에서 다시 동일한 명령을 실행하면 ID=1인 기존 문서를 새 문서로 대체함.
`John Doe` -> `Jane Doe`로 바뀜

인덱싱할 때 ID는 선택 사항. 지정하지 않으면 es에서 임의 ID 생성함.
아래는 ID 없이 인덱싱하는 예시.
ID 지정하지 않았으므로 `PUT` 대신 `POST` 사용.
```
POST /customer/external?pretty
{
  "name": "Jane Doe"
}
```