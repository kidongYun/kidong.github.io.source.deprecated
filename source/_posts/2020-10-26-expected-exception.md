---
layout: post
title:  "Junit 예외 테스트하는 방법에 대한 고찰"
date:   2020-10-26 11:38:54 +0900
categories: java
---

테스트를 하게 되면 특정 예외를 발생시키는 코드에 대해서도 테스트를 해야할 경우가 있다. 예를 들면 파라미터 검증을 할 때 특정 값이 없다면 예외가 던져지는지 확인해야하고, 외부 연동을 하고 나서 응답 값이 비정상일 때에도 원하는 흐름대로 예외가 던져지는지 확인해야 한다. 

Junit을 활용하여 예외를 테스트하는 방법은 기본적으로 크게 어려운 것이 없어 단순하게 샘플 코드만을 제시했다.

```java

    @Test(expected = Exception.class)
    public void refundFee_PnrIsNull() throws Exception {
        ee.expect(Exception.class);
        ee.expectMessage("PNR 이 없습니다");

        airlineTW.refundFee(null);
    }

```

@Test 어노테이션에는 'expected' 속성이 있으며 해당 속성에 예측되는 예외 클래스 타입을 적어주면 된다. 그러면 테스트 함수 내부에서 지정된 예외 클래스가 던져진다면 테스트는 성공이 되고 그렇지 않다면 실패한다. 

던져지는 예외에 특정 메시지에 대한 여부도 테스트해야 할 경우에는 위와 같이 @Test 어노테이션 만으로는 불가능 하고 @Rule 어노테이션과 ExpectedException 객체를 활용해야 한다. 주의해야할 점은, ExpectedException 와 관련된 소스가 테스트하려는 비지니스 소스보다 상단에 존재해야 한다.

```java

    @Rule
    public ExpectedException ee = ExpectedException.none();

    @Test
    public void refundFee_PnrIsNull() throws Exception {
        /* 이 소스가 테스트하려는 소스보다 위에 작성되어야 한다.*/
        ee.expect(Exception.class);
        ee.expectMessage("PNR 이 없습니다");

        airlineTW.refundFee(null);
    }

```