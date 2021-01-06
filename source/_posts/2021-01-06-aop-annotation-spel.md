---
layout: post
title:  "어노테이션에 값 넘겨받는 방법에 대한 고찰"
date:   2021-01-06 20:00:00 +0900
categories: java
---

Aspect 객체에 동적으로 들어오는 데이터를 넘겨야하는 경우가 생길 때가 있다. 보통 이럴 때에 나는 커스텀 어노테이션을 만들고 이 어노테이션 값에 SpEL 문법을 활용하여 동적으로 들어온 데이터를 넘겨주는데 이 방법을 남겨둔다.

### Aspect 쪽에서 동적 데이터가 필요한 배경

로깅 작업이 서버에 단순히 로그를 남겨둘 때에도 있지만 만약 좀 더 중요한 로그라면 별도의 데이터베이스 테이블을 구성하고 이 곳에 보통 저장해둔다. 이런 로그들은 비지니스 관련하여 중요도가 높은 로그일 경우가 많은데 보통 이런 로그는 후에 로그를 찾을 때 질의를 쉽게 하기 위해서 ID 값을 함께 넣어둔다.

이런 로깅을 AOP로 처리하기 위해서는 ID 값을 파라미터에서 가져와야 하는데 파라미터의 구조는 함수마다 다르기 떄문에 어떤 값이 ID에 해당하는지 Aspect 객체는 알 수가 없다. 그렇기 떄문에 이를 함수쪽에서 어떤 값이 ID인지를 매핑해주어야 하는데 이 때 SpEL 문법을 활용한다.

### 구현

```java

@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExampleLog {
    String id() default "";
}

```

우선 어노테이션을 만든다. 이 어노테이션을 id 값을 가지고 있으며, 여기를 통해서 동적으로 데이터를 넘겨줄 것이다.

```java

@ExampleLog(id = "#id")
public String example1(String id) {
    return id;
}

@ExampleLog(id = "#value")
public String example2(String value) {
    return value;
}

```

이 어노테이션을 활용해서 데이터를 넘겨주는 함수부분이다. 보면은 파라미터가 어떤 이름으로, 어떤 구조로 들어오는지 알 수 없기 때문에 위와 같이 SpEL 문법으로 매핑을 시켜주고 있다.

```java

@Slf4j
@Aspect
@Component
public class ExampleLogAspect {
    @Around(value = "@annotation(ExampleLog)")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        ExampleLog annotation = method.getAnnotation(ExampleLog.class);

        Object id = getDynamicValue(signature.getParameterNames(), joinPoint.getArgs(), annotation.id());

        log.info("ID : " + id);

        // 로그 테이블에 추가

        return joinPoint.proceed();
    }

    public static Object getDynamicValue(String[] parameterNames, Object[] args, String key) {
        ExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();

        for (int i$ = 0; i$ < parameterNames.length; i$++) {
            context.setVariable(parameterNames[i$], args[i$]);
        }

        return parser.parseExpression(key).getValue(context, Object.class);
    }
}

```

Aspect 쪽 코드이다. Around 에 ExampleLog 어노테이션을 주어서 해당 어노테이션이 설정되어 있는 함수들은 이 AOP 가 적용되도록 하였다, 후에는 리플렉션과 SpEL Parser 를 활용하여 ID에 해당하는 값을 찾아낸다. 그 다음엔 로깅 목적으로 사용하는 테이블에 데이터를 추가하면 된다.
