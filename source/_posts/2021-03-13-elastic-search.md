---
layout: post
title:  "Elastic Search"
date:   2021-03-13 00:0054 +0900
categories: java
---


데이터 베이스 -> 로그 스태쉬 -> 엘라스틱 서치 -> 키바나


엘라스틱서치는 jvm 위에서 동작하기 때문에 jdk 8 이상 있어야한다.

엘라스틱서치는 역색인 기능 덕분에 검색의 기능 관점에서는 RDB보다 훨씬 빠르다.

```
Elastic Search              Relational DB
Index                       Database
Type                        Table
Document                    Row
Field                       Column
Mapping                     Schema
```

REST API 데이터를 CRUD 조작이 가능하다.

```
Elastic Search              Relational DB               CRUD
GET                         Select                      Read
PUT                         Update                      Update
POST                        Inster                      Create
DELETE                      Delete                      Delete
```

Elastic Search GET 예제
```
curl -XGET http://localhost:9200/classses
```
localhost:9200 이 부분이 elasticsearch 서버 IP 주소 및 포트이고 그다음 classes 이거가 인덱스

```
curl -XGET http://localhost:9200/classses?pretty
```
뒤에 pretty 속성을 주면 결과가 보기 좋게 반환된다.


인덱스 생성하기 예제 (PUT을 사용했다.)
```
curl -XPUT http://localhost:9200/classes
```

인덱스 제거하기 예제 (DELETE를 사용한다.)
```
curl -XDELETE http://localhost:9200/classes
```

문자열을 직접 넣어서 document 만들기.
```
curl -XPOST http://localhost:9200/classes/class/1/ -d '
{"title" : "Algorithm", "professor", "john"} '
```

파일을 활용해서 document 만들기.
```
curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json
```

기존에 존재하는 document에 필드 추가하기.
```
curl -XPOST http://localhost:9200/classes/class/1/ update -d '
{"doc" : {"unit" : 1}} '
```

스크립트를 활용해서 다큐먼트 수정하기.
```
curl -XPOST http://localhost:9200/classes/class/1/ update -d ‘
{ “script” : “ctx._source.unit += 5” } ’
```

벌크 (여러가지 다큐먼트를 한번에 삽입하는 방법)
```
curl -XPOST http://localhost:9200/_bulk?pretty —data-binary @classes.json
```

매핑 (Schema)
매핑이 없이도 데이터를 넣을 수 있다.
그러나 실무에서 매핑없이 넣는 것은 위험한 일. 자바스크립트를 사용하는 것 대신 타입스크립트를 쓰는것과 동일.

매핑에서 geo_point 는 지도 위의 위치를 의미.
인덱스를 조회해 보면 mapping 영역이 있음 거기를 처리하는 것.

키바나로 시각화를 할때 매핑된 데이터를 활용해야 더 디테일하게 조작이 가능하다.



———————————————

몽고디비 레플리카 서버 + 엘라스틱서치 검색엔진 적용.

엘라스틱서치 간략한 정리.

왜 엘라스틱서치 검색방법이 더 빠른가. -> 역색인 방법.
빅데이터들을 빠르게 검색하고 처리가 가능하다.

엘라스틱서치 만으로 그러면 서비스를 할수 있지 않나.
단점이 있다 단점이 뭐냐면. 

엘라스틱 서치의 search을 알아보자.

```
curl -XGET localhost:9200/basketball/record/_search?pretty
```

URI의 q 파라미터 값을 활용하여 쿼리를 작성할 수 있다.
```
curl -XGET 'localhost:9200/basketball/record/_search?q=points:30&pretty'
```

REQUEST BODY 를 활용해서도 쿼리를 날릴 수 있다.
```
curl -XGET 'localhost:9200/basketball/record/_search -d `{

    "query" : {
        "term" : { "points" : 30 }
    }
}`'
```

REquest body 옵션은 다양한 질의가 가능하다 나중에 찾아보자.

elastic aggregation.

aggs = aggregations

metry aggs -> 산술 어그리게이션.

BUCKET AGGS -> 그룹 바이로 보면 된다.

뭔가 다큐먼트 통합적으로 연산을 할 떄 사용하는 거 같은데.

디비에서 있는 SUM(), MAX() 이런 연산을 지원하는 거 같다.


키바나 설정.

키바나 management 메뉴.

index pattern -> index 설정하는 곳에 엘라스틱서치의 인덱스를 넣는다.

----------------------

엘라스틱 서치 인강.

루씬 기반의 검색 엔진.
자바 기반으로 만들어짐. (JVM 위에서 돈다. jdk8이상 필요)

Elastic stack -> Elaticsearch + Logstash + Kibana

빅데이터 사업쪽에서 많이 사용하나봄.
엘라스틱 개발자가 돈을 잘번대.

클라우드, 온오프라믹스?

엘라스틱 설치

데이터베이스가 있고 이걸 로그스태쉬를 활용해서 엘라스틱서치에 연동을 할 수 있다.

postgresql -> logstash -> elasticsearch -> kibana
이거 구축을 해보자.



————————————————

오픈소스 검색 및 분석 엔진.
방대한 양의 데이터를 신속하게, 거의 실시간으로 저장, 검색, 분석할 수 있도록 지원.

활용 예시
로그 또는 트랜잭션 데이터들을 모아서 의미 있는 데이터화.
두번째 데이터들 빅데이터들을 모아서 의미 있는 데이터화.

엘라스틱 스택.
엘라스틱 서치 + 키바나 + 로그스태쉬

