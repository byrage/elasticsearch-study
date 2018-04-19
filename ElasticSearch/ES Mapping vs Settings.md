ES 2.4 기준 : https://www.elastic.co/guide/en/elasticsearch/reference/2.4/mapping.html#mapping

# Mapping
문서와 문서에 포함 된 필드를 저장하고 색인을 생성하는 방법을 정의하는 프로세스
 - 문자열 필드가 전체 텍스트 필드로 처리 되어야 하는지
 - 숫자, 날짜, 지리적 위치 포함
 - 문서의 모든 필드값이 _all 에 인덱싱 되어야 하는지 여부
 - date 포맷 값
 - 동적으로 추가 된 필드의 매핑을 제어하는 규칙

## Mapping Fields
- Meta-fields : 문서의 메타 데이터 처리 방법을 커스터마이징 ex) \_index, \_type, \_id, \_source
- Fields or properties : 매핑타입은 필드 리스트나 properties를 포함한다. fields = row, properties = 변수 인듯..
> Field : 같은필드에 다른 방식, 다른 목적으로 인덱싱 될 때 유용하다. 멀티 필드에 유용함. 예를 들어 string 필드는 전체 텍스트 검색에 매핑되고, keyword 필드는 소팅이나 어그리게이션에 사용된다.
```
"mappings": {
    "_doc": {
      "properties": {
        "city": {
          "type": "text",
          "fields": {
            "raw": {
              "type":  "keyword"
            }
          }
        }
      }
    }
  }
city.raw : city 필드의 keyword
city : 풀 텍스트 검색
city.raw : 소팅 + 어그리게이션
```
> 여러가지 방법으로 analyze 하기 위해 멀티 필드를 사용하기도 한다. See [Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html#_multi_fields_with_multiple_analyzers)
> properties : 서브필드를 포함하는 타입 매핑, object 필드, nested 필드. datatype이나 object, nested이다.

- Field datatypes
 - string, data, long, double, boolean, ip
 - object, nested 같은 json 계층을 지원함
 - geo_point, geo_shape, completion같은 특별한 타입
- string을 전체 텍스트 검색을 위해 'analyzed' 필드로, 정렬 또는 집계를 위해 'not_analyzed'필드로 인덱싱 할 수 있다.

## Dynamic Mapping
- 다이나믹 매핑을 이용하면 사용되기 전에 필드와 매핑타입을 정의할 필요 없다.
- 인덱스 이후로 자동으로 새로운 매핑타입과 필드명이 추가될 것이다.
- 새로운 필드가 top-level 매핑타입과 object, nested 필드 안에 자동으로 추가될 것이다.
- 다이나믹 매핑 룰은 새로운 타입과 필드 매핑에 사용되게 커스터마이징 설정될 수 있다.

## Explicit mappings
- 다이나믹 매핑이 초심자에게는 좋을 수 있으나, 어떤 때에는 명시적 매핑이 필요하다.
- 인덱스 생성시에 매핑타입과 필드 매핑을 설정할 수 있고, PUT mapping API를 이용하여 존재하는 인덱스에 추가 할 수도 있다.

## Updating existing mappings
- documented 되면 매핑 필드와 타입은 업데이트 될 수 없다.
- 매핑 필드가 변경 되는 것은 기존의 인덱싱된 도큐먼트가 invalid해 지는 것.
- 대신에 인덱스를 새로 생성하고 재색인 해야 한다.

## Fields are shared across mapping types
- 매핑 유형은 필드를 그룹화하는데 사용되지만 각 매핑 타입의 필드는 서로 독립적이지 않다.
- 타이틀필드가 user, blogpost 매핑타입 내부에 존재한다면, 타이틀 필드는 각각의 타입안에 정확하게 매핑되어야 한다.

## Example
```
{
  "mappings": {
    "user": {
      "_all":       { "enabled": false  },
      "properties": {
        "title":    { "type": "string"  },
        "name":     { "type": "string"  },
        "age":      { "type": "integer" }
      }
    },
    "blogpost": {
      "properties": {
        "title":    { "type": "string"  },
        "body":     { "type": "string"  },
        "user_id":  {
          "type":   "string",
          "index":  "not_analyzed"
        },
        "created":  {
          "type":   "date",
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
```
