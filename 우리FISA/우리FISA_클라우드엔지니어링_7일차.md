# [FISA 클라우드 엔지니어링 7일차]

![화면 캡처 2024-07-16 091749.png](%5BFISA%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A1%E1%84%8B%E1%85%AE%E1%84%83%E1%85%B3%20%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%8C%E1%85%B5%E1%84%82%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%207%E1%84%8B%E1%85%B5%E1%86%AF%E1%84%8E%E1%85%A1%5D%205e81f81b82874f06be0f313310721074/%25ED%2599%2594%25EB%25A9%25B4_%25EC%25BA%25A1%25EC%25B2%2598_2024-07-16_091749.png)

- Beat → Logstash → elasticsearch → kibana

트래블 도메인

HRD 기획부 김성준 추천?

### 엘라스틱 서치 특징

- NoSQL
- 역파일 구조
- document는 하나의 고객
- 인덱스 → 도큐먼트
- 엘라스틱 head tools → brower → 인덱스 이름 클릭하면 테이블로 볼 수 있음
- post _bult 대용량 데이터 삽입
- index 색인 시 keyword, text로 나뉜다.
    - text는 형태소 분석으로 어근마다 분리하여 색인한다.
    - keyword는 검색의 정확도를 위해서 단어 자체를 색인하도록한다. (Atomic 형태소 분석기)

### 키바나 특징

- 매트릭 집계 기능, 시각화

### 엘라스틱 서치에 대용량 CSV 업로드

![화면 캡처 2024-07-16 142355.png](%5BFISA%20%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A1%E1%84%8B%E1%85%AE%E1%84%83%E1%85%B3%20%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%8C%E1%85%B5%E1%84%82%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%207%E1%84%8B%E1%85%B5%E1%86%AF%E1%84%8E%E1%85%A1%5D%205e81f81b82874f06be0f313310721074/%25ED%2599%2594%25EB%25A9%25B4_%25EC%25BA%25A1%25EC%25B2%2598_2024-07-16_142355.png)

- Machine Learning → Max file size 수정

### 키바나로 대시보드 만들기

- **Machine Learning** → **Data Visualizer → Import data (index pattern 생성 확인 후)**
- **Visualize → New visualization → Lens 차트 생성**

### Agreegation 예제 Kibana devtools Json 문법

```java
GET bank/_search
{
  "size": 0,
  "aggs": {
    "all_customers": {
      "sum": {
        "field": "customers"
      }
    }
  }
}
```

### ElasticSearch 자주 사용하는 문법 리스트

```json
# Term Query: 특정 필드의 값과 정확히 일치하는 문서를 찾습니다
GET /empall/_search
{
  "query": {
    "term": {
      "job": "manager"
    }
  }
}

# 특정 필드에 대한 전체 텍스트 검색을 수행합니다.

GET /empall/_search
{
  "query": {
    "match": {
      "description": "software developer"
    }
  }
}

# 특정 범위 내의 값을 가진 문서를 찾습니다.
GET /empall/_search
{
  "query": {
    "range": {
      "salary": {
        "gte": 50000,
        "lte": 100000
      }
    }
  }
}

# 복합 조건을 위한 논리 조합을 제공합니다.
GET /empall/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "job": "developer" } },
        { "range": { "experience": { "gte": 5 } } }
      ],
      "filter": [
        { "term": { "status": "active" } }
      ]
    }
  }
}

# 특정 필드의 고유 값 빈도 계산
GET /empall/_search
{
  "size": 0,
  "aggs": {
    "job_titles": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      }
    }
  }
}

# 특정 필드 값의 평균 계산
GET /empall/_search
{
  "size": 0,
  "aggs": {
    "average_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}

# Date Histogram Aggregation: 날짜 필드를 기준으로 히스토그램 생성
GET /empall/_search
{
  "size": 0,
  "aggs": {
    "employees_over_time": {
      "date_histogram": {
        "field": "hire_date",
        "calendar_interval": "year"
      }
    }
  }
}

# Filter Context: 필터링을 위해 특정 조건에 맞는 문서만 선택
GET /empall/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "department": "engineering" } }
      ]
    }
  }
}

# Scripted Field: 스크립트를 사용하여 동적 필드 생성
GET /empall/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "double_salary": {
      "script": {
        "source": "doc['salary'].value * 2"
      }
    }
  }
}

# Highlighting: 검색 결과의 특정 부분을 하이라이트
GET /empall/_search
{
  "query": {
    "match": {
      "description": "software"
    }
  },
  "highlight": {
    "fields": {
      "description": {}
    }
  }
}
```