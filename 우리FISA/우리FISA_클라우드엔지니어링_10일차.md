# [fisa 클라우드엔지니어링 10일차 (project)]

발표 5분 브리핑

1. 환경설정 기록
2. 실패 - 에러상황도 기록
    1. 성공 - 진행 상황 기록
3. 기록
    1. [readme.md](http://readme.md) or notion or blog

### filebeat → elasticsearch stream

- 날것의 데이터만 들어옴

```json
{
"_index": "filebeat-7.11.1-2024.07.19-000001",
"_type": "_doc",
"_id": "V7pxyJABb8cVgMbfOjkg",
"_version": 1,
"_score": 1,
"_source": {
"@timestamp": "2024-07-19T00:42:59.420Z",
"ecs": {
"version": "1.6.0"
},
"host": {
"ip": [
"fe80::4228:67f2:9332:c5df"
,
"192.168.0.56"
,
"fe80::b99b:6aaf:e941:d7f0"
,
"192.168.56.1"
,
"fe80::b8a7:f11c:70bd:42f0"
,
"169.254.99.155"
,
"fe80::b274:befb:65f:92b9"
,
"169.254.9.148"
,
"fe80::ef29:8baf:b38d:22b5"
,
"169.254.250.222"
],
"name": "2-35",
"mac": [
"c0:18:50:63:0a:a6"
,
"0a:00:27:00:00:0a"
,
"ac:74:b1:4b:b5:1d"
,
"ac:74:b1:4b:b5:1e"
,
"ae:74:b1:4b:b5:1d"
],
"hostname": "2-35",
"architecture": "x86_64",
"os": {
"kernel": "10.0.19041.4648 (WinBuild.160101.0800)",
"build": "19045.4651",
"platform": "windows",
"version": "10.0",
"family": "windows",
"name": "Windows 10 Pro"
},
"id": "83f01c1b-e24d-4369-9edc-e31873cc6032"
},
"log": {
"offset": 260,
"file": {
"path": "C:\00.dataSet\bankfisa3.csv"
}
},
"message": "20200818,NH농협은행,4호점,강남,5724",
"input": {
"type": "log"
},
"agent": {
"id": "0bccbb1c-dc98-4b26-9e9f-129c10c87634",
"name": "2-35",
"type": "filebeat",
"version": "7.11.1",
"hostname": "2-35",
"ephemeral_id": "0326788e-0b6c-488e-b959-36b5d4a09904"
}
}
}
```

- 로그 스테이시의 필요성을 느낌
- filebeat은 실시간으로 csv 파일을 감시하고 elasticsearch index에 데이터를 업데이트함.

### 로그 스테이시 전처리

- filebeat → ES에서 filebeat → logstash로 filebeat.conf 파일 변경
- logstash conf 파일 설정 (전처리)

```yaml
input {
 # beat에서 데이터를 받을 port지정
  beats {
    port => 5044 
  }
}

filter {
  mutate {
    # 실제 데이터는 "message" 필드로 오기 때문에 csv형태의 내용을 분할하여 새로운 이름으로 필드를 추가 
    # 20180601,NH농협은행,1호점,종각,2314
    split => [ "message",  "," ] 
    add_field => {
      "date" => "%{[message][0]}"
      "bank" => "%{[message][1]}"
      "branch" => "%{[message][2]}"
      "location" => "%{[message][3]}"
      "customers" => "%{[message][4]}"
    }

    # 기본으로 전송되는 데이터 분석에 불필요한 필드는 제거한다. "message" 필드도 위에서 재 가공 했으니 제거
    # head에서 보고 불필요한 속성들 삭제 리스트에 저장해서 관리
    remove_field => ["ecs", "host", "@version", "agent", "log", "tags",  "input", "message"]
  }

  # "date" 필드를 이용하여 Elasticsearch에서 인식할 수 있는 date 타입의 형태로 필드를 추가
  date {
    match => [ "date", "yyyyMMdd"]
    timezone => "Asia/Seoul"
    locale => "ko"
    target => "date"
  }

  # Kibana에서 데이터 분석시 필요하기 때문에 숫자 타입으로 변경
  mutate {
    convert => {
      "customers" => "integer"
    }
    remove_field => [ "@timestamp" ]
  }
}

output {

  # 콘솔창에 어떤 데이터들로 필터링 되었는지 확인
  stdout {
    codec => rubydebug
  }

  # 위에서 설치한 Elasticsearch 로 "bank" 라는 이름으로 인덱싱 
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "bankfisa3"
  }
}
```

- filebeat 종료 후 로그스테이시 conf 파일 설정
- 이후 로그스테이시 bin에서 설정한 conf 파일지정 후 실행
- beat → logstash → ES
- filebeat은 이미 읽은 파일에 대해서 cache하고 있기 때문에 이미 filebeat → ES로 연결했던 파일은 filebeat → Logstash → ES로 새롭게 파이프라인을 만드려면 다른 파일명으로 읽게 만들어야한다.

 

### Window에서 ELK 스택구축 단계

> 1단계 - jdk 실행 환경 수정
- ES 정상실행, logstash 실행 시 jdk 17은 지원하지 않는 문제발생
- 해결책 : logstash의 jdk 경로 설정
- 방식 : cmd창 관리자 모드
- 셋팅 후 종료 재실행 후 로그스테이시 재실행

2단계 - 실행순서
es → logstash → filebeat
es/bin:es.bat
logstash/bin : logstash
filebeat : filebeat

3단계 - 실행을 위한 선행 단계
csv 파일에 데이터 저장 - filebeat은 이미 read했던 파일은 파일명 수정해도 새로운 데이터로 간주하지 않음. 새로운 파일 만들어야함.
filebeat.yml 수정 (새로운 파일 경로 설정)
- logstash/conf/*.conf 파일로 전처리 로직 적용 filebeat으로부터 요청 받는 port 설정, es에 index명 설정 후 output 지정

4단계 - 실행 명령어
es - 단순 실행
logstash → bin → logstash -f ../config/*.conf 
filebeat → filebeat -e -c filebeat.yml

5단계 - 확인 및 test
- 브라우저 head에서 생성 또는 이미 존재하는 index명인 경우 새로 추가된 데이터 확인
- csv 파일에 새로운 데이터 추가 저장 후 콘솔창 확인
- 브라우저 head에서 데이터 확인
> 

### Window에서 설치한 것 Linux로 옮겨보기

### 리눅스에서 엘라스틱 서치 설치 후 실행하려고 하는데 다음과 같은 에러 발생

./elasticsearch-env: line 77: /home/username/ELK/elasticsearch-7.11.1/jdk/bin/java: cannot execute binary file: Exec format error

arm이 아닌 86_64로 엘라스틱서치 운영체제 호환성 파일 변경

### 리눅스에서 로그 스테이시 실행 시 다음과 같은 에러 발생

./logstash -f -d /home/username/ELK/logstash-7.11.1/config/bank.conf
Using bundled JDK: /home/username/ELK/logstash-7.11.1/jdk
./logstash: line 63: /home/username/ELK/logstash-7.11.1/jdk/bin/java: cannot execute binary file: Exec format error
./logstash: line 63: /home/username/ELK/logstash-7.11.1/jdk/bin/java: Success

자바 11버전 설치 후 .profile 수정 후 source 이후 로그스테이시 재실행

### filebeat를 실행하려고 하는데 다음과 같은 에러 발생

./filebeat -e -c -d filebeat.yml
Error: unknown command "filebeat.yml" for "filebeat"
Run 'filebeat --help' for usage.

-d 옵션은 사용할 수 없음

ssh로 파일을 업로드 한거라. chmod go-w 설정 해줘야함

### 정상적으로 파일이 변환되어서 엘라스틱에 저장됐지만 호스트에서 접속이 불가’

elasticseatch.yml 파일 0.0.0.0 으로 수정 후 클러스터 1개만 사용하도록 설정

[https://m.blog.naver.com/itso-/222115351087](https://m.blog.naver.com/itso-/222115351087)

- 운영체제에 기본으로 설정되어 있는 mmap이 65530으로 너무 적게 설정되어 있어 생기는 오류
- 그리고 가상메모리 확장해야함.

### 엘라스틱서치 재실행 하려고 했는데 시스템 디스크 용량 부족문제발생

sudo rm -rf /var/tmp/*
sudo rm -rf /tmp/*
sudo apt-get clean

sudo find /var/log -type f -delete

sudo apt-get autoremove
sudo apt-get autoclean

### filebeat → logstash → ELK

파일빗으로 파일 감지 후 실시간으로 데이터 업데이트 확인

### 총정리

1. 엘라스틱 서치 conf 설정
    1. 운영체제 최대 메모리값 설정
    2. 0.0.0.0 으로 설정하고 싶다면 싱글노드 설정도 함께 해주기
    3. 엘라스틱 서치 실행
2. 로그스테이시 conf 설정
    1. bank.conf 파일 input, fileter, ouput 로직작성
    2. input : filebeat, output: EK
    3. 로그스테이시 실행
        1. ./logstash -f ../config/bank.conf
3. 파일빗 conf 설정
    1. /home/username/ELK/filebeat-7.11.1-linux-x86_64/filebeat.yml
    2. path 설정 : csv 파일경로 설정하기 : /home/username/ELK/data/bank2.csv
    3. filestream false 설정하기
    4. 파일빗 실행
    5. ./filebeat -e -c filebeat.yml