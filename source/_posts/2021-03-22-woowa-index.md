---
layout: post
title:  "INDEX"
date:   2021-03-22 00:0054 +0900
categories: java
---

INDEX - 임의의 규칙대로 부여된 임의의 대상을 가리키는 무언가.

SEQ 번호 같은 개념.

### Clusted vs Non-Clustered

Cluster : 군집

Clustered : 군집화

Clustered Index : 군집화된 인덱스.

인덱스와 데이터가 아주 밀접하게 군집되어 있다.

Clustered Index 의 특징
이미 정렬되어 있는 데이터들임

검색할 때는 성능이 좋다.

새로운 데이터들이 들어올 때에는 다른 데이터들을 다 밀거나 넣거나 하기 때문에 추가, 수정하기가 성능이 안좋다.

Clustered Index 는 데이터와 친밀, 정렬되어 있음. 그로인해 발생하는 특징.

Non-Clustered Index 는 인덱스 번호가 데이터를 가지고 있는게 아니고 주소번지를 가지고 있음.

연결리스트 느낌이랄까..

약한 참조 관계다.

보통 인덱스와 실제 데이터 간 주소를 해시테이블로 해서 빠르게 찾을 수 있도록 한다.

PK를 index 처리를 한다면 그것은 보통 clustered index 방식이고 그렇기 때문에 auto_increment 방식을 사용할 수 있음. 만약에 그 중간에 숫자를 넣으려고하면 헬.. PK는 계속 숫자가 증가되기 때문에 괜찮음.

Cardinaltiy -> 인덱스를 평가 요소. 인덱스로 쓰는 값이 중복된다면 카디널리티가 낮아지고. 유일한 값들이라면 카디널리키가 높아진다.

카디널리티가 높을 수록 인덱스로 사용하는 것을 고려해봐야 한다.

email을 PK로 가져갈 경우 DB 성능에 이슈가 있을수도 있다.

PK는 보통 clustered index 방식.
정렬를 해줘야함.
이메일은 새로운 데이터가 들어오면 다시 다 재정렬 해야함.
숫자로 되어있으면 auto_increment.

PK 가 아닌 값으로 검색.