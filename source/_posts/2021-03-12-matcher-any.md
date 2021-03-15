---
layout: post
title:  "Matcher any 여러 개일때와 null 있을 때"
date:   2021-03-12 00:0054 +0900
categories: java
---

Test Code 작성시 Mocking을 위해서 Matcher 클래스는 유용하게 사용되는데. 파라미터가 여러 개 들어갈 경우. 실제 값과 any를 활용한 값이 섞여서 전달되면
정상적으로 Mocking이 되지 않는다.

두번째로 null 객체를 넣고 싶을 때에는 isNull(T) 이 함수를 활용하면 된다.

```java
        when(mypageServiceMock.retrieve(isNull(PartnerVO.class), anyString(), anyString(), anyString(), anyString(), anyString(), anyBoolean(), Matchers.<BookingMapper>any(), anyString()))
                .thenReturn(retrieveDetail);
```