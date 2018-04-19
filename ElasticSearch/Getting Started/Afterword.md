1. NRT에 대해서 더 알아보자
 - 별 내용 안나온다..
 - 기본적으로 index/update/delete후에 바로 적용되는것이 아니고 검색이 가능하기 까지 1초 정도 걸린다. 이 정도 내용이면 될듯
1. ES에서 INSERT가 INDEXING이랑 동의어인가?
 - POST / PUT 메소드를 이용하여 documents를 insert하는데
 - insert 행위의 대상이 index임. index에 데이터가 들어가면 indexing이 되기 때문에 조금은 다른 측면인듯
1. Shard라는 용어가 인덱스를 나누는것 까지인가.. 나누고 배치하는 것 까지인가? 옵션으로 테스트 해보자
 - 기본적으로 ES 옵션은 Shards = 5, Replication = 1 임
 - 싱글 노드로 index를 생성했을 때 index가 5개로 나뉘고, replication 1세트의 데이터가 대기 중인 것으로 보아서 shards는 데이터 분리까지인듯
1. 샤드를 수평으로 split, scale하는 것?
 - http://d2.naver.com/helloworld/14822
 - 주민 테이블을 정자동, 서현동으로 나누어 같은 스키마를 가진 다른 db에 배치 = split
 - 나눈 db를 replicas로 복제하는 것 = scale
1. ES Shards == Lucene Index?
 - 그렇다고는 하는데 특별한 내용들은 없음..
1. ES PUT vs POST 정리
 - https://discuss.elastic.co/t/when-to-use-put-versus-post/68550/2
 - https://en.wikipedia.org/wiki/Representational_state_transfer - Relationship between URL and HTTP methods 참조
 - POST로 update를 하려면 /customer/\_doc/1/\_update 식으로 고유한 keyword를 사용함
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html
1. indices?
 - index의 복수형... indexes는 3인칭 단수란다
 - ngram viewer에서 검색해보니 indices가 indexes 보다 더 많이 나오긴 하네
