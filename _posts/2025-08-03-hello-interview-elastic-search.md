---
title: "Elasticsearch 엘라스틱서치 deep dive"
categories:
  - study
tags:
  - System Design
  - elasticsearch
---

## Elasticsearch Deep Dive

### Introduction

Elasticsearch란?
* 가장 인기 있고 강력한 오픈 소스 분산 검색 엔진
* 수십 년간 구축된 방대한 프로젝트: 시스템 디자인 문제에서 검색 및 검색과 관련된 많은 문제를 해결하는 데 사용
* 대부분의 데이터베이스 시스템(예: Postgres의 full text search)도 검색에 능숙하지만, 특정 규모나 정교함 수준에서는 목적에 맞게 구축된 시스템이 필요함.
* distributed, asynchronous, and concurrent
* 정렬(sorting), 필터링(filtering), 랭킹(ranking), 패싯(faceting) 등 다양한 요구사항을 처리함.

### 검색의 기본 개념
* Criteria: 검색을 위해 지정하는 조건이나 규칙
  * 제목에 특정 단어 포함
  * 가격 범위
* Facet: 검색 결과를 더 세분화하고 정제할 수 있게 도와주는 옵션 - 처음 입력한 검색 기준을 바탕으로 나온 결과들을 특정 기준에 따라 분류하여 보여줌으로써, 사용자가 원하는 정보를 더 쉽게 찾을 수 있도록 돕는 기능
* Results: 검색의 결과 - Set of Documents

### Elasticsearch 기본 개념 및 사용법

RESTful API를 제공하여 쉽게 작업을 수행할 수 있음.

#### Basic Concepts
![image](https://github.com/user-attachments/assets/f09d352a-8753-4bf9-931f-887277687bb6)
* 문서 (Documents): 검색 대상이 되는 개별 데이터 단위. JSON 객체 같은 것.
```json
{ 
    "id": "XYZ123", 
    "title": "The Great Gatsby", 
    "author": "F. Scott Fitzgerald", 
    "price": 10.99, 
    "createdAt": "2024-01-01T00:00:00.000Z" 
}
```
* 인덱스: 문서의 컬렉션. 관계형 데이터베이스의 '데이터베이스 테이블'과 유사
  * 각 문서는 고유한 ID와 검색할 데이터를 포함하는 키-값 쌍인 필드(fields)와 연결됨.
  * 검색은 인덱스를 대상으로 수행되며, 검색 기준과 일치하는 문서 결과를 반환합니다.
* 매핑 및 필드 (Mappings and Fields):
  * 매핑(Mapping): 인덱스의 스키마. 인덱스가 가지는 필드와 각 필드의 데이터 유형, 그리고 필드가 처리되고 인덱싱되는 방식과 같은 다른 속성을 정의.
  * 매핑은 어떤 필드가 검색 가능하고 어떤 타입의 데이터를 포함하는지 Elasticsearch에 알려주므로 매우 중요. - 사진에서 `price: Float`
  * 타입은 복잡할 수 있으며, 중첩 객체, 배열, 지리 공간 유형, 사용자 정의 분석기 또는 시맨틱 검색을 위한 임베딩을 사용할 수 있습니다.
  * `keyword` 타입은 전체 값으로 처리되어 효율적인 검색 및 정렬에 유용(hash table)하며, `text` 타입은 토큰화되어 텍스트 검색에 사용됨 (reverse index).
  * 매핑에 사용하지 않는 많은 필드를 포함하면 인덱스의 메모리 오버헤드가 증가하여 성능 문제 및 비용 증가.

#### Basic Use
* 인덱스 생성: 간단한 `PUT` 요청으로 동적 매핑, 1개의 샤드, 1개의 복제본을 가진 인덱스를 생성.
```
PUT /books 
{
    "settings": { 
    "number_of_shards": 1, 
    "number_of_replicas": 1 
    } 
}
```
 * 매핑 설정: 데이터의 대부분 필드가 검색 가능하지 않거나, Elasticsearch가 필드 유형을 정확히 추론하기 어렵다면 미리 매핑을 설정할 수 있음. (This lets Elasticsearch know that certain fields should be treated as searchable and what types to expect in those fields.) 이제 book을 추가하면 Elasticsearch가 정확한 값을 추출해서 index를 만들어 검색될 수 있게 할 것.
```
PUT /books/_mapping 
{
  "properties": {
    "title": { "type": "text" },
    "author": { "type": "keyword" },
    "description": { "type": "text" },
    "price": { "type": "float" },
    "publish_date": { "type": "date" },
    "categories": { "type": "keyword" },
    "reviews": {
      "type": "nested",
      "properties": {
        "user": { "type": "keyword" },
        "rating": { "type": "integer" },
        "comment": { "type": "text" }
      }
    }
  }
}
```
  * 중첩 필드(Nested Fields): `reviews`와 같이 중첩된 문서를 정의할 수 있음. 이는 데이터 업데이트 및 쿼리 패턴에 따라 결정다. (예: 리뷰가 자주 업데이트되지 않고 자주 쿼리되는 경우 중첩이 효율적).
* 문서 추가 (Add Documents):
```
// POST /books/_doc
{
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "description": "A novel about the American Dream in the Jazz Age",
  "price": 9.99,
  "publish_date": "1925-04-10",
  "categories": ["Classic", "Fiction"],
  "reviews": [
    {
      "user": "reader1",
      "rating": 5,
      "comment": "A masterpiece!"
    },
    {
      "user": "reader2",
      "rating": 4,
      "comment": "Beautifully written, but a bit sad."
    }
  ]
}
```
  * 각 요청은 문서 ID와 클러스터에 문서가 유지된 방법에 대한 데이터를 반환
```
{
  "_index": "books",
  "_id": "kLEHMYkBq7V9x4qGJOnh",
  "_version": 1, // NOTE!
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
  * `_version` 필드는 문서가 원자적으로 업데이트되도록 보장하는 데 사용됩니다.
* 문서 업데이트 (Updating Documents)
  * 문서 ID를 URL에 지정하여 전체 문서를 `PUT`할 수 있지만, 동시성 문제가 발생할 수 있음: A가 읽음 -> B가 읽음 -> A가 씀 -> B가 씀
  * `_version` 필드를 사용하여 버전이 일치하는 경우에만 업데이트하도록 지정하여 낙관적 동시성 제어(optimistic concurrency control)를 구현 가능. `PUT /books/_doc/kLEHMYkBq7V9x4qGJOnh?version=1`
  * `_update` 엔드포인트(`POST`)를 사용하여 문서의 특정 필드만 업데이트
```
// POST /books/_update/kLEHMYkBq7V9x4qGJOnh
{
  "doc": {
    "price": 14.99
  }
}
```

* 검색 (Search)
  * Elasticsearch의 쿼리 문법은 SQL과 유사하며 JSON 기반
  * `GET /books/_search` 엔드포인트를 사용하며, `query` 객체 안에 다양한 쿼리 타입을 사용하여 필터링
```
// GET /books/_search
{
  "query": {
    "match": {
      "title": "Great"
    }
  }
}
```
    * match: 특정 필드에 특정 단어가 포함된 문서.
    * bool (must, should, filter, must_not 등): 여러 조건을 조합하여 검색.
    * range: 숫자 또는 날짜 범위로 검색.
    * nested: 중첩된 필드 내에서 검색.
```
// GET /books/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Great" } },
        { "range": { "price": { "lte": 15 } } }
      ]
    }
  }
}
```
```
// GET /books/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "bool": {
          "must": [
            { "match": { "reviews.comment": "excellent" } },
            { "range": { "reviews.rating": { "gte": 4 } } }
          ]
        }
      }
    }
  }
}
```
  * 검색 결과는 `took`, `timed_out`, `_shards`, `hits` 등의 정보를 포함하며, `hits` 배열에는 일치하는 문서와 `_score` (관련성 점수)가 포함.
```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 2.1806526,
    "hits": [
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "1",
        "_score": 2.1806526,
        "_source": {
          "title": "The Great Gatsby",
          "author": "F. Scott Fitzgerald",
          "price": 12.99
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "2",
        "_score": 1.9876543,
        "_source": {
          "title": "Great Expectations",
          "author": "Charles Dickens",
          "price": 10.50
        }
      }
    ]
  }
}
```
* 정렬 (Sort)
  * 기본 정렬: `sort` 파라미터를 사용하여 검색 결과를 특정 필드를 기준으로 정렬. (다중 정렬도 가능)
```
// GET /books/_search
{
  "sort": [
    { "price": "asc" },
    { "publish_date": "desc" }
  ],
  "query": {
    "match_all": {}
  }
}
```
  * 스크립트 기반 정렬: [Painless Script Language](https://www.elastic.co/docs/explore-analyze/scripting/modules-scripting-painless)를 사용하여 계산된 값으로 정렬
```
// GET /books/_search
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "source": "doc['price'].value * 0.9"
        },
        "order": "asc"
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```
  * 중첩 필드 (Nested Fields) 정렬: 중첩된 객체 내의 값으로 정렬할 때 `nested` 정렬을 사용.
```
// GET /books/_search
{
  "sort": [
    {
      "reviews.rating": {
        "order": "desc",
        "mode": "max",
        "nested": {
          "path": "reviews"
        }
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}
```
  * 관련성 기반 정렬: 정렬 순서를 지정하지 않으면 Elasticsearch는 관련성 점수(`_score`)에 따라 결과를 정렬. 기본 알고리즘은 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)와 밀접하게 관련되어 있음

* 페이지네이션 및 커서 (Pagination and Cursors)
  * From/Size 페이지네이션: 가장 간단한 형태, `from` (시작 인덱스)과 `size` (반환할 결과 수)를 지정. - 깊은 페이지네이션(예: 10,000개 이상)에는 비효율적 (due to the overhead of sorting and fetching all preceding documents. The cluster needs to retrieve and sort all these documents on each request, which can be prohibitively expensive.)
```
// GET /my_index/_search
{
  "from": 0,
  "size": 10,
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  }
}
```
  * Search After: 깊은 페이지네이션에 더 효율적. 
  * 이전 페이지의 마지막 결과의 정렬 값을 다음 페이지의 시작점으로 사용.
    * 첫 쿼리는 그냥 보냄.
    * 쿼리의 결과에서 sort value를 저장.
    * 다음 쿼리를 보낼 때 search after 필드의 위 value들을 함께 보냄.
    * 그 사이에 새로운 document가 추가되어도 중복으로 데이터를 보여주지 않음. (처음부터 20개씩 보여주는 방식이면 중간에 데이터가 새로 생겨서 페이지가 넘어가는 document들은 두번 보여주게 됨)
    * 클라이언트 측에서 상태를 유지해야 하며, 페이지에 대한 무작위 접근은 허용되지 않음.
```
// GET /my_index/_search
{
  "size": 10,
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  },
  "sort": [
    {"date": "desc"},
    {"_id": "desc"}
  ],
  "search_after": [1463538857, "654323"]  // 직전 페이지 마지막 document의 timestamp와 id
}
```
> [!NOTE]
> 데이터 변경으로 인한 누락: 만약 사용자가 한 페이지를 보고 다음 페이지를 요청하는 사이에 인덱스의 기본 데이터(문서)가 삭제되거나 업데이트되어 정렬 순서가 변경된다면, 이전에 반환되었어야 할 문서가 다음 페이지에서는 나타나지 않거나 아예 누락될 수 있습니다.
> 예를 들어, 책 목록을 정렬하여 첫 번째 페이지(1~10번)를 받았다고 가정해봅시다. 사용자가 두 번째 페이지를 요청하기 직전에, 7번째에 있던 책이 삭제되거나 *가격이 바뀌어* 정렬 순서가 크게 변경될 수 있습니다. 이 경우, 두 번째 페이지를 요청하면 11번째로 나타났어야 할 책이 갑자기 10번째로 당겨지거나, 심지어 11번째 이후에 있던 책이 10번째 자리를 채울 수 있어, 사용자는 일관된 순서로 모든 결과를 보지 못하고 특정 문서가 누락된 것처럼 느낄 수 있습니다. 이는 검색 결과가 결과적 일관성(eventual consistency) 모델을 따르기 때문에 발생할 수 있는 현상입니다.
>        ▪ 첫 페이지 결과: [책 A (5달러), 책 B (7달러), 책 C (10달러)]
>        ▪ 다음 페이지를 위한 search_after 값은 **책 C의 정렬 값(10달러)**이 됩니다.
>    ◦ 이제 사용자가 다음 페이지를 요청하기 전에 '책 D (원래 12달러)'가 '8달러'로 가격이 변경되었다고 가정해 봅시다.
>        ▪ 원래 정렬 순서: 책 A (5), 책 B (7), 책 C (10), 책 D (12)...
>        ▪ 가격 변경 후 정렬 순서: 책 A (5), 책 B (7), 책 D (8), 책 C (10)...
> 이러한 문제를 해결하기 위해 Elasticsearch는 Point in Time (PIT) API와 search_after를 함께 사용하는 방식을 제공합니다. PIT는 특정 시점의 인덱스 스냅샷을 생성하여, 그 스냅샷을 기준으로 페이지네이션을 진행함으로써 데이터 변경에 관계없이 일관된 결과를 제공할 수 있습니다

  * 커서 (Cursors): `point in time` (PIT) API와 `search_after`를 함께 사용하여 페이지네이션 전반에 걸쳐 데이터의 일관된 뷰(consistent view)를 제공.
    * PIT ID를 생성(`POST /my_index/_pit?keep_alive=1m`)하고, 검색 쿼리에 PIT ID를 포함하며, `search_after`와 함께 사용.
```
// GET /_search
{
  "size": 10,
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  },
  "pit": {
    "id": "46To...",
    "keep_alive": "1m"
  },
  "sort": [
    {"_score": "desc"},
    {"_id": "asc"}
  ]
}
```
    * 작업 완료 시 PIT를 닫아야 함. 서버 자원 소모가 크기 때문에 `keep_alive`를 지정할 것.

```
// DELETE /_pit
{
  "id" : "46To..."
}
```

### Elasticsearch 작동 원리

Elasticsearch는 Apache Lucene이라는 고도로 최적화된 검색 라이브러리의 오케스트레이션 프레임워크. [트위터 같은 곳에서는 Lucene를 직접 사용합니다.](https://gayuna.github.io/system%20design/twitter-design-in-real/)

* Elasticsearch의 역할: 클러스터 조정, API, 집계, 실시간 기능 등 분산 시스템 측면을 처리.
* Lucene의 역할: 검색 기능 자체를 담당. Lucene은 단일 노드에서 작동하며, Elasticsearch는 클러스터 수준에서 작동합니다.

#### 클러스터 아키텍처

Elasticsearch는 분산 검색 엔진이며, 여러 노드를 실행. 하나의 물리적 머신이 여러 노드 유형의 책임을 맡을 수도 있음.

##### 노드 유형
* 마스터 노드: 클러스터를 조정하는 역할. 노드 추가/제거, 인덱스 생성/삭제와 같은 클러스터 수준 작업을 수행할 수 있는 유일한 노드. 클러스터당 하나의 활성 마스터 노드만 존재.
* 데이터 노드: 데이터를 저장하는 역할. 실제 데이터가 저장되는 곳. 큰 클러스터에서는 많은 데이터 노드를 가짐. 데이터 노드는 `hot`, `warm`, `cold`, `frozen`과 같이 데이터 접근 빈도에 따라 전문화될 수 있음.
* 코디네이팅 노드: 클러스터 전반의 검색 요청을 조정하는 역할. 클라이언트로부터 검색 요청을 수신, 적절한 노드로 전송.
* 인제스트 노드: 데이터 수집을 담당. 데이터가 색인 전에 변환되고 준비되는 곳.
* 머신러닝 노드: 머신러닝 작업을 담당합니다.

##### 노드 간 상호작용
![Image](https://github.com/user-attachments/assets/e2aef54e-0a85-42bd-9e7e-f222d66e62b1)
인제스트 노드가 데이터 노드에 데이터를 로드하고, 코디네이팅 노드를 통해 쿼리하는 방식으로 작동.
각 노드 유형의 책임은 다른 하드웨어 요구 사항을 내포합니다 (예: 데이터 노드는 높은 디스크 I/O 또는 더 많은 메모리가 필요).
When the cluster starts, you'll specify a list of seed nodes (these are master-eligible) which will perform a leader election algorithm to choose a master for the cluster. Only one node should be the active master at a time, while the other master-eligible nodes are on standby.

#### 문서 수집 및 검색 흐름

1.  문서 수집:
  * 클라이언트가 인제스트 노드에 문서를 보냅니다.
  * 인제스트 노드는 수집 파이프라인을 실행하여 문서를 처리한 후 데이터 노드로 전달합니다.
  * 데이터 노드가 문서를 수신하고 인덱싱하면 인제스트 노드에 확인 응답을 보내고, 이는 클라이언트에게 다시 확인됩니다.
  * 데이터 노드에서는 문서가 Lucene 인덱스에 추가되며, 대부분의 경우 새 세그먼트가 생성됩니다.
2.  문서 검색:
  * 클라이언트가 코디네이팅 노드에 연결합니다.
  * 코디네이팅 노드는 사용자가 제출한 쿼리를 구문 분석하고, 데이터 접근 계획(어떤 샤드에 상주하는지 등)을 세웁니다.
  * 계획에 따라 관련 데이터 노드에 쿼리를 제출합니다.
  * 데이터 노드는 쿼리를 Lucene 인덱스에 전달하며, Lucene 인덱스는 관련 세그먼트에 걸쳐 병렬로 실행됩니다.
  * 역인덱스와 Doc Values 같은 기능을 활용하여 결과를 필터링하고 정렬합니다.
  * 데이터 노드는 부분적인 결과를 코디네이팅 노드에 반환하고, 코디네이팅 노드는 이 결과들을 병합하여 사용자에게 반환합니다.
* 병렬 처리: 이 과정은 여러 수준에서 병렬 처리가 이루어집니다 (예: 여러 샤드 동시 쿼리, 다른 복제본으로 요청 전송, 인덱스 내에서 여러 CPU에 걸쳐 작업 병렬화).

#### 데이터 노드

데이터 노드의 주요 기능은 문서를 저장하고 빠르게 검색 가능하게 하는 것.
![image](https://github.com/user-attachments/assets/1b0016a0-c316-47a0-a476-1544d2684455)
* _source 데이터 vs. Lucene 인덱스: Elasticsearch는 원본 `_source` 데이터와 검색에 사용되는 Lucene 인덱스를 분리.
* 요청 처리 단계
  1. 쿼리 단계 (Query Phase): 최적화된 인덱스 데이터 구조를 사용하여 관련 문서를 식별
  2. 가져오기 단계 (Fetch Phase): (선택적으로) 식별된 문서 ID를 노드에서 가져옴
  * 이상적인 쿼리는 소스 문서를 전혀 건드리지 않고 답변할 수 있는 쿼리입니다.
* 인덱스, 샤드, 복제본 (Indices, Shards, Replicas):
  * 데이터 노드는 인덱스(문서의 컬렉션)를 포함, 인덱스는 샤드(shards)와 그 복제본(replicas)으로 구성.
  * 샤드:
    * Elasticsearch가 데이터(및 관련 인덱스)를 여러 호스트에 분할할 수 있게 해주는 기능
    * 문서와 해당 인덱스 구조를 클러스터 내 여러 노드에 분산시켜 성능과 확장성을 크게 향상
    * 문서는 상호 배타적으로 하나의 샤드에 할당되며, 검색은 모든 관련 샤드에서 병렬로 실행되고, 결과는 조정 노드에 의해 병합 및 정렬
    * 샤드에는 Lucene 인덱스가 캡슐화되어 있음
  * 복제본 (Replica):
    * 샤드의 정확한 복사본 (exact copy)
    * Elasticsearch는 인덱스의 샤드 사본을 하나 이상 생성할 수 있으며, 이를 replica shards 또는 그냥 replicas이라고 함
    * 일반적으로 고가용성(high availability)과 처리량 증가(increased throughput)를 위해 도입
    * 코디네이팅 노드는 모든 사용 가능한 샤드 사본(기본 샤드 및 복제본)에 검색 요청을 분산하여 검색 성능을 향상시키고 검색 워크로드를 클러스터 전체에 효과적으로 분산
  * Elasticsearch shards are 1:1 with Lucene indexes

#### Lucene 세그먼트 CRUD (Lucene Segment CRUD)

Lucene 인덱스는 검색 엔진의 기본 단위인 세그먼트(segments)로 구성.

* 불변성: 세그먼트는 색인된 데이터의 불변(immutable) 컨테이너.
  * 쓰기 작업은 배치로 처리되며 세그먼트를 구성. 문서를 삽입할 때 즉시 인덱스에 저장되는 것이 아니라 세그먼트에 추가됨. 문서 배치가 모이면 세그먼트를 구성하고 디스크로 플러시.
  * 세그먼트 수가 너무 많아지면 병합할 수 있음. 병합하려는 세그먼트로부터 새 세그먼트를 생성하고 이전 세그먼트를 제거.
  * 삭제: 각 세그먼트에는 삭제된 식별자 집합이 있고, 삭제된 문서에 대한 데이터를 쿼리할 때 해당 데이터는 존재하지 않는 것처럼 처리되지만, 데이터는 여전히 세그먼트에 남아 있음. 병합 작업 중에 삭제된 문서가 정리됨.
  * 업데이트: 실제 세그먼트 자체를 업데이트하지 않음. 대신, 이전 문서를 soft delete하고 업데이트된 데이터로 새 문서를 삽입. 이전 문서는 나중에 세그먼트 병합 이벤트에서 정리.
    * 이로 인해 삭제는 매우 빠르지만, 병합 및 정리 전까지는 일부 성능 저하가 발생할 수 있음.
    * 업데이트는 삽입보다 성능이 낮음. 이는 Elasticsearch가 빠르게 업데이트되는 데이터에 적합하지 않은 이유 중 하나.
* 불변 아키텍처의 장점:
  * 향상된 쓰기 성능: 새 문서를 기존 세그먼트 변경 없이 새 세그먼트에 빠르게 추가할 수 있음.
  * 효율적인 캐싱: 세그먼트는 불변이므로 일관성 문제 없이 메모리나 SSD에 안전하게 캐시될 수 있음
  * 단순화된 동시성: 읽기 작업은 쿼리 도중 데이터 변경에 대해 걱정할 필요가 없어 동시 접근이 단순화
  * 쉬운 복구: 충돌 시, 불변 세그먼트는 상태가 알려져 있고 일관성이 유지되므로 복구가 더 쉬움.
  * 최적화된 압축: 불변 데이터는 더 효과적으로 압축되어 디스크 공간을 절약.
  * 더 빠른 검색: 불변성으로 인해 검색을 위한 최적화된 데이터 구조와 알고리즘이 가능
* 주기적인 세그먼트 병합의 필요성 및 정리 작업 전 일시적으로 증가하는 저장 공간 요구사항과 같은 과제도 있음

#### Lucene 세그먼트 특징

세그먼트는 검색 작업에 관련된 고도로 최적화된 데이터 구조를 담고 있습니다.

* 역인덱스 (Inverted Index)
  * Lucene의 핵심은 역인덱스.
  * 콘텐츠(예: 단어 또는 숫자)에서 데이터베이스의 위치(문서)로의 매핑을 저장하는 데 사용되는 데이터 구조
  * Elasticsearch를 사용한 키워드 검색을 빠르게 함.
  * 모든 문서에 나타나는 고유한 단어를 나열하고 각 단어가 나타나는 모든 문서를 식별.
  * "great"이라는 단어를 찾는다 가정: 모든 문서를 스캔하는 대신, 역인덱스를 찾아 문서 ID를 O(1)에 찾을 수 있음.
  * You can copy your data and organize the copy like (1) (?)
* Doc Values
  * 역인덱스가 토큰에서 문서로의 매핑을 제공한다면, Doc Values는 최종 정렬을 수행하는 데 필요한 데이터를 제공.
  * on-disk
  * 정렬들을 수행할 때는 역인덱스와 달리 특정 컬럼의 값이 필요하게 됨.
  * Sorting, aggregations, and access to field values in scripts requires a different data access pattern. Instead of looking up the term and finding documents, we need to be able to look up the document and find the terms that it has in a field.
  * 관계형 데이터베이스와 같은 행 지향 데이터베이스의 문제점인, 단일 컬럼에만 접근해야 할 때 전체 행을 읽어야 하는 비효율성을 해결함.
  * The doc_values field is an on-disk data structure that is built at document index time and enables efficient data access. It stores the same values as _source, but in a columnar format that is more efficient for sorting and aggregation.
  * Doc Values 구조는 세그먼트의 모든 문서에 대해 단일 필드의 컬럼형, 연속적인 표현(contiguous representation)을 가짐 -> 빠른 정렬 및 필터링을 가능하게 합니다.

#### 코디네이팅 노드

코디네이팅 노드는 클라이언트로부터 요청을 받아 클러스터 전반에 걸쳐 실행을 조정하는 역할을 합니다.

* 쿼리 플래너: 검색 쿼리를 실행하는 가장 효율적인 방법을 결정하는 알고리즘
  * 쿼리 플래너는 쿼리 실행 계획을 평가하여 관련 문서를 검색하는 최상의 방법을 결정
  * 여기에는 역인덱스 사용 여부 결정, 쿼리 부분 실행 순서 결정, 여러 노드에서 온 결과 결합 조정 등이 포함
  * 예를 들어, "bill nye"를 검색할 때, "nye"가 포함된 문서 수가 "bill"보다 훨씬 적다면, "nye"가 포함된 문서를 먼저 찾아 교집합을 구하는 것이 훨씬 효율적
  * Elasticsearch의 쿼리 플래너는 필드의 유형, 인기 있는 키워드, 문서 길이 등 통계를 유지하여 사용자에게 결과를 반환하는 데 걸리는 시간을 최소화하도록 최적의 옵션을 선택
  * 이러한 최적화는 데이터 세트의 크기와 복잡성이 증가함에 따라 성능을 유지하는 데 중요
  * 데이터 의존성(data dependence)을 처리할 수 있게 하여, 시스템이 인덱스의 데이터에 동적으로 반응할 수 있게 함

### 인터뷰에서의 활용

#### Elasticsearch 사용 시 고려사항

* 주요 데이터베이스로 사용하지 말 것: Elasticsearch는 기본적으로 검색 엔진이며, 기존 데이터베이스를 대체하기 위한 것이 아님
  * 일반적으로 PostgreSQL 또는 DynamoDB와 같은 권한 있는 데이터 스토어에 Change Data Capture (CDC)를 통해 연결
* 읽기 중심 워크로드에 최적화됨: 쓰기 중심 시스템을 다루는 경우 다른 옵션을 고려하거나 쓰기 버퍼를 구현
  * 잦은 업데이트는 Elasticsearch의 성능을 저하시킴
* 최종 일관성 모델 허용: 결과는 지연될 수 있으며, 경우에 따라 상당히 오래 걸림
  * 강력한 일관성이 필요한 경우에는 다른 데이터베이스를 사용해야 함
* 데이터 비정규화(Denormalization): 검색 쿼리를 효율적으로 만들기 위해 데이터를 최대한 비정규화해야 함
  * Elasticsearch는 관계형 데이터베이스가 아님: 조인(JOINs)을 지원하지 않으며, 단일 또는 두 개의 쿼리로 결과를 제공해야 함
* 항상 필요한 것은 아님: 데이터가 작거나(10만 문서 미만) 자주 변경되지 않는다면, 더 간단하고 빠른 솔루션
  * 기본 데이터 스토어에 대한 간단한 쿼리가 충분한지 확인하고, 부족할 경우에만 Elasticsearch를 고려
* 데이터 동기화 유지: 기본 데이터와 Elasticsearch 간의 동기화를 주의 깊게 유지해야 함. 동기화 실패는 드리프트(drift)를 유발하고 일반적인 버그의 원인이 됨.
* 선택 정당화: Elasticsearch가 강력한 도구이지만 만능 해결책은 아니므로, 다른 옵션보다 Elasticsearch를 선택하는 이유를 정당화하고 장점뿐만 아니라 한계도 논의할 준비를 해야 함.

#### Elasticsearch에서 얻을 수 있는 시스템 디자인 팁

1. 불변성(Immutability)의 힘: 스택의 적절한 계층에서 불변성을 사용하면 캐싱, 압축 및 데이터 최적화 능력을 향상시킬 수 있습니다. 또한 가변 데이터에서 해결하기 훨씬 어려운 동기화 및 무결성 문제를 걱정할 필요가 없습니다.
2. 쿼리 실행과 데이터 저장 분리: 쿼리 실행을 데이터 저장소에서 분리함으로써 각 부분을 독립적으로 최적화할 수 있습니다. Elasticsearch의 데이터 노드와 코디네이팅 노드는 각 노드 유형의 책임에 집중함으로써 서로를 완벽하게 보완합니다.
3. 인덱싱 전략의 중요성: 인덱싱 전략은 검색 성능에 큰 영향을 미칩니다. Elasticsearch의 역인덱스 구조는 빠른 전문 텍스트 검색을 가능하게 하고, Doc Values는 효율적인 정렬 및 집계를 가능하게 합니다. 빠른 데이터 검색이 필요한 시스템을 설계할 때 가장 일반적인 쿼리 패턴을 지원하기 위해 데이터를 어떻게 구조화할지 고려해야 합니다.
4. 분산 시스템의 복잡성: 분산 시스템은 확장성과 내결함성을 제공하지만, 복잡성도 증가시킵니다. Elasticsearch의 클러스터 아키텍처는 대량의 데이터와 높은 쿼리 부하를 처리할 수 있지만, 데이터 일관성 및 네트워크 분할에 대한 신중한 고려도 필요합니다. 분산 시스템을 설계할 때 항상 일관성, 가용성, 분할 허용 오차(CAP 이론) 사이의 절충점을 고려해야 합니다.
5. 효율적인 데이터 구조의 중요성: Elasticsearch가 역인덱스를 위해 스킵 리스트(skip lists) 및 유한 상태 변환기(finite state transducers)와 같은 전문화된 데이터 구조를 사용하는 것은 특정 사용 사례에서 맞춤형 데이터 구조가 성능을 극적으로 향상시킬 수 있음을 보여줍니다. 데이터 구조를 선택하거나 설계할 때 항상 데이터의 접근 패턴을 고려해야 합니다.
