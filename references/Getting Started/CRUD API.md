## List All Indicies

### GET /\_cat/indices?v
```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```
- 아직 indices에 아무것도 없어서 간단하게 나옴

## Create an index
curl -XPUT 'localhost:9200/customer?pretty&pretty'
curl -XGET 'localhost:9200/\_cat/indices?v&pretty'

- 첫 번째 커맨드는 **PUT** 메소드로 customer index **껍데기** 생성. **pretty** 를 뒤에 붙임으로 JSON response를 pretty-print로 보여달라고 요청함

> pretty vs non-pretty

```
curl -XPUT 'localhost:9200/customer?pretty&pretty'

{
  "error" : {
    "root_cause" : [
      {
        "type" : "resource_already_exists_exception",
        "reason" : "index [customer/g5Rl_Q7FSS2PAltW6XF6Ng] already exists",
        "index_uuid" : "g5Rl_Q7FSS2PAltW6XF6Ng",
        "index" : "customer"
      }
    ],
    "type" : "resource_already_exists_exception",
    "reason" : "index [customer/g5Rl_Q7FSS2PAltW6XF6Ng] already exists",
    "index_uuid" : "g5Rl_Q7FSS2PAltW6XF6Ng",
    "index" : "customer"
  },
  "status" : 400
}

curl -XPUT 'localhost:9200/customer'

{"error":{"root_cause":[{"type":"resource_already_exists_exception","reason":"index [customer/g5Rl_Q7FSS2PAltW6XF6Ng] already exists","index_uuid":"g5Rl_Q7FSS2PAltW6XF6Ng","index":"customer"}],"type":"resource_already_exists_exception","reason":"index [customer/g5Rl_Q7FSS2PAltW6XF6Ng] already exists","index_uuid":"g5Rl_Q7FSS2PAltW6XF6Ng","index":"customer"},"status":400}%
```

```
curl -XGET 'localhost:9200/_cat/indices?v&pretty'

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```

- customer라는 1개의 인덱스와 5개의 primary(origin) shards, 1개의 replica, 0개의 document가 있음을 확인 가능
- status가 Yellow임도 확인 가능. 이것은 아까보았듯이 replicas가 아직 allocated 되지 않은 것인데, 그 이유는 ES에서 디폴트로 한개의 replica를 생성했기 때문.
- 현재 하나의 노드에서 실행중이기 때문에 아직 allocated되지 않은 것이고, (고가용성을 위해) 추후에 다른 노드가 cluster에 join해서 allocated 한다면 헬스는 Green이 될 것이다.

## Index and Query a Document
새로운 customer document를 Index안에 생성해보자
```
curl -XPUT 'localhost:9200/customer/_doc/1?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'

{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

- customer index안에 documents가 잘 생성 되었음을 볼 수 있음
- 내부적으로 1이라는 ID를 받은 것을 확인 가능
- Elasticsearch에서는 먼저 문서를 색인화하기 전에 먼저 색인을 명시 적으로 작성하지 않아도된다는 점에 유의해야합니다. 이전 예에서 Elasticsearch는 사전에 이미 존재하지 않는 고객 색인을 자동으로 작성합니다. -> 무슨뜻이지? index를 생성하지 않아도 document를 넣으면 그 타입에 맞게 index도 만들어준다는 뜻인가
> 테스트해보니 ** PUT /customer**랑 **PUT /customer/\_doc/1** 의 차이. 앞에 것은 껍데기만 생성하고, 뒤에거는 껍데기 + 데이터도 생성

```
<REST Verb> /<Index>/<Type>/<ID>

요거만 기억하면 ES의 API 커맨드를 사용할 수 있음. 잘 기억하라고 함
```

## Modifying Your Data

- ES는 Data Manipulation과 near real time 검색을 제공함
- 기본적으로 index/update/delete후에 검색 결과로 나오기 까지 1초 정도의 딜레이를 기대할 수 있다.
- 이것은 sql같은 플랫폼에서는 트랜잭션 완료 후에 즉시 접근가능한 것과 중요한 구별이다.

### Indexing/Replacing Documents
customer/1로 John Doe를 create 한 후에 Jane Doe로 변경하는 예제
```
curl -XPUT 'localhost:9200/customer/_doc/1?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```
```
curl -XPUT 'localhost:9200/customer/_doc/1?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'
```
- 다른 ID로 입력했으면 replace가 아닌 new document가 index 되었을 거라고 함
- Indexing할때 Id는 옵셔널이다. id를 주지 않으면 random ID를 generate할 것이라고 함.
- ID를 주든 주지않든 response로 id를 return 해줄 거라고함

```
curl -XPOST 'localhost:9200/customer/_doc?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'

{
  "_index": "customer",
  "_type": "_doc",
  "_id": "A3N_E2IBygFAW6xpsEPI",
  "_version": 1,
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
- PUT 대신에 POST를 사용하고 id를 부여하지 않으니 랜덤한 id가 나옴.
> PUT은 ID가 필요하고 POST는 OPTIONAL이다.

## Updating documents
- ES에서 업데이트는 실제로는 기존 문서를 삭제 한다음, 업데이트가 적용된 새문서의 Index를 생성함

```
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```
- 이름을 변경하면서 age라는 field를 등록 할 수도 있음

```
POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```
- script를 사용한 예제. age를 5 더함
- ctx.\_source는 업데이트 될 현재 source document를 참조함
- ES는 Query Condition을 이용한 멀티플 documents Update를 제공함. (= SQL Update-Where). See [Update-by-Query](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/docs-update-by-query.html)

## Deleting documents
```
DELETE /customer/_doc/2?pretty
```
- See [Delete-By-Query](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/docs-delete-by-query.html)
- 전체문서를 삭제할때는 그냥 전체 삭제 하는것이 delete-by-query를 사용한 전체삭제보다 효율적이다.

## Batch Processing
- ES는 [\_bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/docs-bulk.html)를 index, update, delete를 batch로 수행 가능
- 이 기능은 네트워크 request-response를 가능한 빠르게 여러개의 작업을 수행할 때 매우 효율적인 메커니즘이다.
- Bulk API는 하나의 액션이 실패해도 실패하지 않는다. 만약 하나의 액션이 어떤이유에서던 실패해도 잔여 액션을 그 이후에 계속함. Bulk API가 리턴될 때 각각의 액션에 대한 status를 제공할것이다. 그러니까 거기서 특정한 액션이 실패했는지 아닌지를 잘 확인해라

```
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```
- id 1,2 생성

```
POST /customer/_doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```
- 1 업데이트, 2 삭제

## Exploring Your Data
- 이제 간단한 기능은 한번 훑어봤으니, 좀더 현실적인 데이터로 해보자.

### Loading the Sample Dataset
- [account.json](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)을 다운받고 다음 커맨드를 실행.

```
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/account/_bulk?pretty&refresh" --data-binary "@accounts.json"
curl "localhost:9200/_cat/indices?v"

health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb
```

- 1000개의 documents가 bank로 bulk indexed 되었다는 의미

## The Search API
- Search에는 2가지 방법이 있음.
  1. 검색 파라미터를 request URI로 보내는 방식
  1. 검색 파라미터를 request body로 보내는 방식
- Body로 보내는 방식은 좀더 표현력이 강해지고, JSON 포맷으로 읽기 쉽게 정의 가능

```
GET /bank/_search?q=*&sort=account_number:asc&pretty

{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

#### REQUEST
- \_search 엔드포인트로 bank를 검색
- q=* 파라미터로 매치되는 모든 document를 지시함
- sort=account_number:asc 파라미터로 오더링
- pretty로 pretty-printed JSON

#### response
- took : 서치하는데 걸린시간 (ms)
- timed_out : 서치하는데 timeout 했는지 boolean
- \_shards : 몇개의 shards를 서치했는지, 검색한 것들 중에 성공/실패 샤드 개수
- hits : 검색 개수
- hits.total : 검색 개수 전체
- hits.hits : 실제로 검색 결과 개수 (디폴트로는 처음 10개)
- hits.sort : 결과의 sort key (score로 소팅했으면 없음)

```
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```
- 위에서 검색한 쿼리의 Request Body 버전

- 검색 결과를 이해하는 것은 중요하다. ES는 request를 완료하고 검색 결과를 저장하지 않고, 너의 결과에 cursor를 오픈하지 않는다. -> 일회성, 휘발성 이라는 얘기인듯
- SQL에서는 원한다면 stateful한 server-side 커서를 가져올 수 있는데 ES에는 없다.
