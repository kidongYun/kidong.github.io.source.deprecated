---
layout: post
title:  "Spring Thymeleaf"
date:   2021-07-02 00:0054 +0900
categories: Spring
---

Thymeleaf 만의 특징은 다른 템플릿 엔진과 다르게 Html Tag를 그대로 사용한다는 것입니다. 

```html
// Thymeleaf 예시
<p th:each="user : ${users}" th:text="${user.name}"></p>
```