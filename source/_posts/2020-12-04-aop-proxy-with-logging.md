---
layout: post
title:  "로깅을 활용한 Proxy, AOP 에 대한 고찰"
date:   2020-12-04 11:38:54 +0900
categories: spring
---

실무에서 코드를 작성하게 되면 로그를 남기는 행위는 굉장히 범용적으로 필요한 일이다. 어느 특정 비지니스에 해당하지 않고 모든 곳에 사용이 되기 때문이다. 가독성의 관점에서 이 로깅을 하는 코드들은 비지니스 코드에 의존되지 않고 영향이 가지 않도록 작성하는 것이 중요하다. 이 글에서 이를 해결하기 위한 고민해보록 하자.

### 가장 간편한 로깅 방법

```java

@Slf4j
class Calculator {
    public int add(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 + value2;
        long endTime = System.currentTimeMillis();

        log.info("value1 : " + value1 + ", value2 : " + value2);
        log.info("result : " + result);
        log.info("execute time : " + (endTime - startTime) + " ms");
        return result;
    }
}

```

이 글에서 구현하려고하는 로깅의 기능은 한 함수의 범위에서 들어오는 입력 파라미터의 값들과, 리턴되는 결과 값을 보여주고 해당 함수의 실행시간을 보여주는 기능이다. 위에서 제시한 로깅 방법은 이를 가장 간단하게 구현한 방법이다. 이번에는 덧셈의 기능을 하는 add() 뿐만 아니라 뺄셈과 곱셈의 기능을 하는 sub(), mul() 도 구현하고, 로깅 기능도 넣어보도록 하자.

```java

@Slf4j
class Calculator {
    public int add(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 + value2;
        long endTime = System.currentTimeMillis();

        log.info("value1 : " + value1 + ", value2 : " + value2);
        log.info("result : " + result);
        log.info("execute time : " + (endTime - startTime) + " ms");
        return result;
    }

    public int sub(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 + value2;
        long endTime = System.currentTimeMillis();

        log.info("value1 : " + value1 + ", value2 : " + value2);
        log.info("result : " + result);
        log.info("execute time : " + (endTime - startTime) + " ms");
        return result;
    }

    public int mul(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 * value2;
        long endTime = System.currentTimeMillis();

        log.info("value1 : " + value1 + ", value2 : " + value2);
        log.info("result : " + result);
        log.info("execute time : " + (endTime - startTime) + " ms");
        return result;
    }
}

```

함수가 여러 개가 되니까 간편하게 구현한 로깅기능에 문제점 하나가 보이는 것 같다. 모두 동일하게 입력되는 파라미터와, 반환되는 결과 값을 보여주는 로그인데 같은 기능을 함에도 소스가 중복이 되고 있다. 이를 개선하기 위해 첫번째로 생각해 볼수 있는 방법은 Logging을 위한 객체를 별도로 두고 이 객체를 호출하도록 하는 것이다. 여기서 로깅 작업의 목표는, 파라미터, 리턴값, 실행시간이기 때문에 이를 위한 값을 파라미터로 받는다.

```java

@Slf4j
class Logger<T, U> {
    public void execute(T params, U returnValue, long startTime, long endTime, String methodName) {
        log.info("Params : " + params.toString());
        log.info("Method Name : " + methodName);
        log.info("Return Value : " + returnValue.toString());
        log.info("Execute Time : " + (endTime - startTime) + " ms");
    }
}

@Slf4j
class Calculator {
    private final Logger<List<Integer>, Integer> logger;

    public Calculator() {
        logger = new Logger<>();
    }

    public int add(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 + value2;

        logger.execute(List.of(value1, value2), result, startTime,
                System.currentTimeMillis(), Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }

    public int sub(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 + value2;

        logger.execute(List.of(value1, value2), result, startTime,
                System.currentTimeMillis(), Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }

    public int mul(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = value1 * value2;

        logger.execute(List.of(value1, value2), result, startTime,
                System.currentTimeMillis(), Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }
}

```

Logging을 위해서 Logger라는 객체를 생성했고 파라미터, 리턴값, 시작시간, 종료시간, 메소드명을 받도록 했다. Logger 객체를 활용해서 로깅을하게 되면 모두 Logger 객체에서 로깅을 하게됨으로 어떤 객체에서 이 로깅을 한것인지를 알 수 없기 때문에 메서드 명 항목을 추가했다.

위 소스를 보면서도 느끼지만 로깅을 비지니스 영역에서 빼기 위해서 굉장히 난잡해진 코드를 작성했다. 소스의 중복은 사라졌다고 하지만 비즈니스 영역에서 로깅 작업을 제외하는 이유는 가독성이 큰데 사실 가독성이 그렇게 증가한 것 같지도 않다. 또 보시면 알겠지만 startTime 값은 비즈니스 영역의 함수를 호출하기 전 생성이 되어야 하기 때문에 logger 객체에 바로 넣지 못하고 결국 변수를 하나 생성했다.

로깅을 전담하는 객체를 만들어서 이 객체를 활용해 로깅을 하는 위와같은 방식은 근본적으로 비즈니스 소스에서 로깅 객체를 호출하는 구조를 가지고 있다. 즉 로깅 작업을 없애려고 노력했지만 결국 한줄이라도 로깅 객체를 호출하는 부분이 남아 있다. 우리는 비즈니스 영역의 코드와 로깅을 위한 코드의 완벽한 분리를 원한다. 이러려먼 어떻게 해야할까?

이번에 도전해볼 방법은 프로그래밍 디자인 패턴 중 프록시 패턴을 활용할 것이다. 이 패턴은 우리가 원하는 작업처럼 비즈니스 소스와 그 외의 소스를 구분시킬 때 사용한다. Calculator 객체를 감싸는 새로운 객체를 하나 더 만들고, 그 객체에서 Calculator 객체의 비즈니스 코드를 호출하는 것이다. 이렇게 하면 다른 클라이언트 소스에서는 Calculator 를 직접 호출하지 않고 이 프록시 객체를 호출하게 되면 로깅 작업도 정상 작동한다.

위에서 적용한 로거 객체를 만드는 방법과 프록시 객체를 생성하는 방법은 사실 완전 다른데, 로거 객체를 만드는 방법의 호출 관계는 비즈니스 코드가 더 범위가 넓고 이 안에서 로깅 코드를 호출하는 것이였다면, 프록시 객체를 만드는 방법은 반대로 비즈니스 코드를 감쌀 수 있는 더 넓은 범위의 객체를 생성하는 것이다. 이 관점에서 로깅 객체를 만드는 것과는 다른 관점에서의 해결 방법이다.

```java

@Slf4j
class Logger<T, U> {
    public void execute(T params, U returnValue, long startTime, long endTime, String methodName) {
        log.info("Params : " + params.toString());
        log.info("Method Name : " + methodName);
        log.info("Return Value : " + returnValue.toString());
        log.info("Execute Time : " + (endTime - startTime) + " ms");
    }
}

@Slf4j
class Calculator {
    private final Logger<List<Integer>, Integer> logger;

    public Calculator() {
        logger = new Logger<>();
    }

    public int add(int value1, int value2) {
        return value1 + value2;
    }

    public int sub(int value1, int value2) {
        return value1 + value2;
    }

    public int mul(int value1, int value2) {
        return value1 * value2;
    }
}

class CalculatorProxy {
    private final Calculator calculator;
    private final Logger<List<Integer>, Integer> logger;

    public CalculatorProxy() {
        this.calculator = new Calculator();
        this.logger = new Logger<>();
    }

    public int add(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = calculator.add(value1, value2);
        long endTime = System.currentTimeMillis();

        logger.execute(List.of(value1, value2), result, startTime, endTime,
                Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }

    public int sub(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = calculator.sub(value1, value2);
        long endTime = System.currentTimeMillis();

        logger.execute(List.of(value1, value2), result, startTime, endTime,
                Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }

    public int mul(int value1, int value2) {
        long startTime = System.currentTimeMillis();
        int result = calculator.mul(value1, value2);
        long endTime = System.currentTimeMillis();

        logger.execute(List.of(value1, value2), result, startTime, endTime,
                Thread.currentThread().getStackTrace()[1].getMethodName());

        return result;
    }
}

```

Calculator 객체 괸점에서는 비즈니스 영역의 코드만 남았고, CalculatorProxy 객체에서 로그를 남기는 코드를 작성하고 있다. 우리가 원하는 목표는 이루었지만 무언가 깔끔하지 않다. 바로 CalculatorProxy 객체의 문제이다. 이 객체는 Calculator 객체에 다른 함수가 생길 때 마다 CalculatorProxy 객체에도 그 함수를 위한 함수를 생성해줘야하는 문제가 발생한다.

AOP와 OOP 개념은 서로 적대적인 개념이 아니며 상호 보완적인 개념이다. 현재 위 소스들은 Calculator 라는 객체 중심의 사고를 하면서 만들어지고 있다. add(), sub(), mul() 이러한 함수들도 Calculator 객체가 가지는 동작들 중 일원이며, 함수들의 정의가 객체 중심적으로 이루어지고 있다. 객체를 중심으로 코드를 작성하게 되면 위에서 우리가 만들어 낸것 처럼 중복의 업무가 발생할 수 있다.

로깅 작업은 실무에서 대부분의 동작에 필연적으로 들어가야하는 공통 소스이다. 그렇기 때문에 이를 OOP 방식으로 소화하려고 하면 로깅 관련 소스의 중복이 어쩔 수 없다. 여기서 AOP 방식을 활용하면 이를 보다 심플하게 개선할 수 있다. AOP는 이런 로깅 작업처럼 객체 관점의 코딩으로 보았을 때 중복이 많아지거나 하는 문제점이 생기는 코드들을 해결하기 위한 방법이다.

동작원리는 위에서 우리가 활용한 Proxy 객체와 유사하며 이러한 객체를 만들고 객체들의 비즈니스 코드가 동작되기 이전, 이후에 호출되어 로깅등과 같은 작업을 한다.

```java

@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExecuteLog {
}

@Slf4j
@Component
@Aspect
public class ExecuteLogAspect {
    @Around(value = "@annotation(ExecuteLog)")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long end = System.currentTimeMillis();

        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        log.info("Method Name : " + method.getName());
        log.info("Input : " + Arrays.toString(signature.getParameterNames()));
        log.info("Output : " + result.toString());
        log.info("Execute Time : " + (end - start) + " ms");

        return result;
    }
}

```

이 코드는 Spring 에서 제공하는 AOP 구현 방식이다. 위와 같이 코드를 작성하면 @ExecuteLog 어노테이션을 붙인 함수들이 동작하기 전, 후에 위에 작성한 함수가 실행이 된다. 동작원리는 위에서 언급한 프록시 객체가 런타임시에 스프링이 자동으로 결합하여 생성한다. 이것에 대한 상세 원리는 들여다 보지 않아서 잘 모르겠지만 느낌은 이와 비슷하다.

AOP를 하는 방법은 스프링에서 제공해주는 방법뿐만 아니라 다양하게 있다. 또 스프링은 런타임시에 프록시 객체를 만들지만 다른 방법들은 JVM 구동시에 혹은 클래스파일 생성할 때 만들어지는 것도 있다. 