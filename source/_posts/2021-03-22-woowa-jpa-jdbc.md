---
layout: post
title:  "DTO vs VO"
date:   2021-03-22 00:0054 +0900
categories: java
---

DTO = Data Transfer Object.

레이어 간 데이터를 전달하는 객체

로직을 가지지 않은 순수한 데이터 객체.
getter/setter 메서드만을 가진다...?

VO

값 그 자체를 표현하는 객체.

서로 다른 이름을 가진 VO의 인스턴스가 모든 속성 값이 같다면 같은 객체이다.
(equals(), hashcode() 오버라이딩)

객체의 불변성을 보장한다.

로직을 포함할 수 있다.

