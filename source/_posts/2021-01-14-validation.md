---
layout: post
title:  "Validation"
date:   2021-01-14 11:38:54 +0900
categories: spring
---

```
implementation("org.springframework.boot:spring-boot-starter-validation")  
```

mockMVc 에서 controllerAdvice 테스트하는 방법

```java
mockMvc = MockMvcBuilders.standaloneSetup(signControllerMock).setControllerAdvice(new Advice()).build();
```
