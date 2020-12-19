---
layout: post
title:  "Junit 예외 테스트하는 방법에 대한 고찰"
date:   2020-10-26 11:38:54 +0900
categories: java
---

테스트를 하게 되면 특정 예외를 발생시키는 코드에 대해서도 테스트를 해야할 경우가 있다. 예를 들면 파라미터 검증을 할 때 특정 값이 없다면 예외가 던져지는 지 확인해볼 수 있고, 

```java

    @Test(expected = Exception.class)
    public void refundFee_PnrIsNull() throws Exception {
        ee.expect(Exception.class);
        ee.expectMessage("PNR 이 없습니다");

        airlineTW.refundFee(null);
    }

```

but It's not supported a customizing for checking exception messages or errorCode made by users.
for those, we can use @Rule annotation. It's the detail version of @Test(expected = ...)

```java

    @Rule
    public ExpectedException ee = ExpectedException.none();

    @Test
    public void refundFee_PnrIsNull() throws Exception {
        ee.expect(Exception.class);
        ee.expectMessage("PNR 이 없습니다");

        airlineTW.refundFee(null);
    }

```

You don't need to get any assert code but you have to put the code related to ExpectedException at the above part than tested code.
It would be worked well when Exception is occured.