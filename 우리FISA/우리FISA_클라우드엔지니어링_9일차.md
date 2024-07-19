# [FISA 클라우드 엔지니어링 9일차]

### Elasticsearch Text 타입과 Keyword 타입의 차이

```java
PUT string_index
{
  "mappings": {
    "properties": {
      "data1": {
        "type": "text"
}, "data2": {
        "type": "keyword"
      }
} }
}

```

- data1은 text, 2는 keyword로 인덱스 생성

```java

GET string_index/_search
{
  "query": {
    "match": {
        "data1": "pink red blue"
    }
}
```

- text 타입인 data1으로 검색 시 데이터 검색됨

```java
GET string_index/_search
{
  "query": {
    "match": {
        "data2": "pink"
    }
} }
```

- data2로 검색 시 검색불가 hit 값이 들어있지 않음

### 실행결과

```java
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
},
"hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.8630463,
    "hits" : [
      {
        "_index" : "string_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8630463,
        "_source" : {
          "data1" : "pink red blue",
          "data2" : "pink red blue"
        }
			} 
		]
	} 
}

{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
},
"hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
} }
```

- text는 “pink”, “red”, “blue” 단어 단위로 형태소 분석기가 작동하고
- keyword는 정확한 단어 “pink red blue” 로 검색해야 검색된다.

```java
GET my_index/_search
{
  "query": {
    "match": {
    "message": "pink blue"
    }
} }
# match_phrase() : 구절 단위로 검색
# 검색어 순서도 중요
# keyword 단위로 간주
# slop : match phrase API에서 검색하는 단어 사이트의 특정 단어 개수 설정

GET my_index/_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "pink blue",
        "slop": 3
			} 
		}
	} 
}
```

- match : 불순물 허용검색
- match_phrase: 구절단위로 검색(불순물 허용 안함)
- slop : match phrase API에서 검색하는 단어 사이트의 특정 단어 개수 설정하는 옵션 (불순물 개수 허용)

### ELK 스택 Window 환경에서 구축하기

- elasticseartch.bat 실행
- logstash -f ../pipline.conf
- filebeat -e -c filebeat.yml
- 각각 실행 후 filebaet이 읽고있는 csv 파일을 수정하여 실시간으로 데이터를 수집하는지 확인