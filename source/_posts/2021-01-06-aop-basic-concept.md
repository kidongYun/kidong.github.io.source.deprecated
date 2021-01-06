---
layout: post
title:  "AOP 개념에 대한 고찰"
date:   2021-01-06 20:00:00 +0900
categories: java
---

```

1. RTW(Run-Time Weaving) 
    - Spring AOP의 기본 설정이다. proxy 사용 방법으로 Runtime시에 핵심코드와 Advice를 호출한다. 

2. CTW(Compile-Time Weaving)
    - AspectJ 모듈에서 지원되는 방식으로 핵심코드와 Advice를 Compile시에 병합(Merge) 한다.

3. LTW(Load-Time Weaving)
    - AspectJ 모듈에서 지원되는 방식으로 프로세스가 시작될 때 핵심코드와 Advice를 조합한다.
```

스프링 AOP 에서는 런타임 시점에서만 동작을 하며 