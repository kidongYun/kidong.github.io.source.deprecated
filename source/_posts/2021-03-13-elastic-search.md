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
