---
layout: post
title:  "Validation"
date:   2021-01-14 11:38:54 +0900
categories: spring
---

bean validation의 종류가 여러개 있는 것 같은데 그 중 하나가 hibernate validator

기본적으로 Bean Validation은 클래스 필드에 특정 어노테이션(annotation)을 달아 제약 조건을 정의하는 방식으로 동작한다. 그 후 해당 클래스 객체를 Validator를 이용해 제약 조건의 충족 여부를 확인한다.