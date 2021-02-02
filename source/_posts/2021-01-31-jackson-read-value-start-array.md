---
layout: post
title:  "Jackson을 사용해서 배열로 시작하는 Json 문자열을 객체로 변경"
date:   2021-01-31 00:0054 +0900
categories: java
---

단순하게 원하는 객체의 배열형으로 타입을 선언해서 readValue() 함수의 두번째 인수로 주면, 배열 타입으로 json 문자열이 객체화 되어서 반환되는데. 이 값을 Arrays.asList() 함수를 사용해서
List 타입으로 변환할 수 있다.

```java

List<Plan> plans = Arrays.asList(objectMapper.readValue(response, Plan[].class));

```
