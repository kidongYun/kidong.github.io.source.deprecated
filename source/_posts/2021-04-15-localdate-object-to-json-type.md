---
layout: post
title:  "LocalDate json timestamp"
date:   2021-04-15 00:0054 +0900
categories: java
---

```java
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule()); // 추가
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS); // 추가
```