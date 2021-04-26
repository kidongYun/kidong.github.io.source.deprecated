---
layout: post
title:  "Callable 을 람다로 받기"
date:   2021-04-26 00:0054 +0900
categories: java
---

```java
        List<Callable<List<Schedule>>> callables = requests.stream()
                .map(req -> (Callable<List<Schedule>>) () -> search(req)).collect(toList());
```