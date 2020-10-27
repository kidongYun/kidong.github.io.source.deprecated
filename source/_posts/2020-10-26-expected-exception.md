---
layout: post
title:  "Exception test code using JUnit."
date:   2020-10-26 11:38:54 +0900
categories: java
---

You can test the exception simply like the below.

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