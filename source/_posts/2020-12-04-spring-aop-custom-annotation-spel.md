---
layout: post
title:  "로깅을 활용한 Spring AOP에 대한 고찰"
date:   2020-12-04 11:38:54 +0900
categories: spring
---

실무에서 코드를 작성하게 되면 로그를 남기는 행위는 굉장히 범용적으로 필요한 일이다. 어느 특정 비지니스에 해당하지 않고 모든 곳에 사용이 되기 때문이다. 그렇기 때문에 이 로깅을 하는 코드들은 비지니스 코드에 의존되지 않고 영향이 가지 않도록 작성하는 것이 중요하다. 이 글에서 이에 대해서 고민해보록 하자.

### 가장 간편한 로깅 방법

```java

public int add(int value1, int value2) {
    log.info("value1 : " + value1 + ", value2 : " + value2);

    int result = value1 + value2;

    log.info("result : " + result);
    return result;
}

```

이 글에서 구현하려고하는 로깅의 기능은 한 함수의 범위에서 들어오는 입력 파라미터의 값들과, 리턴되는 결과 값을 보여주는 기능이다. 위에서 제시한 로깅 방법은 이를 가장 간단하게 구현한 방법이다. 이번에는 덧셈의 기능을 하는 add() 뿐만 아니라 뺄셈과 곱셈의 기능을 하는 sub(), mul() 도 구현하고, 로깅 기능도 넣어보도록 하자.

```java

public int add(int value1, int value2) {
    log.info("value1 : " + value1 + ", value2 : " + value2);

    int result = value1 + value2;

    log.info("result : " + result);
    return result;
}

public int sub(int value1, int value2) {
    log.info("value1 : " + value1 + ", value2 : " + value2);

    int result = value1 + value2;

    log.info("result : " + result)
    return result;
}

public int mul(int value1, int value2) {
    log.info("value1 : " + value1 + ", value2 : " + value2);

    int result = value * value2; 

    log.info("result : " + result);
    return result;
}

```

함수가 여러개가 되니까 간편하게 구현한 로깅기능에 문제점 하나가 보이는 것 같다. 모두 동일하게 입력되는 파라미터와, 반환되는 결과 값을 보여주는 로그인데 같은 기능을 함에도 소스가 중복이 되고 있다. 