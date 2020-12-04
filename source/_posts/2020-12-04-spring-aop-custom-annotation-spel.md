---
layout: post
title:  "Spring AOP + Custom Annotation + SpEL"
date:   2020-12-04 11:38:54 +0900
categories: spring
---

# 주제 Spring AOP

Spring boot 안쓰고 Spring 으로 할경우 Dependencies 추가해줘야함.

```
	compile group: 'org.aspectj', name: 'aspectjrt', version: '1.9.2'
	compile group: 'org.aspectj', name: 'aspectjweaver', version: '1.9.2'
```

아래와 같이 Aspect 소스를 짰음

```java

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Slf4j
@Component
@Aspect
public class AirlineLog {
    @Around("within(com.air.interpark.ariLine.common.*)")
    public Object loggerAop(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("It's loggerAop");
        return joinPoint.proceed();
    }
}

```

어드바이스 종류는 대따 많음

Around 어드바이스

타겟의 메서드가 호출되기 이전(before) 시점과 이후 (after) 시점에 모두 처리해야 할 필요가 잇는

부가기능을 정의한다.

-> Joinpoint 앞과 뒤에서 실행되는 Advice



Before 어드바이스

타겟의 메서드가 실행되기 이전(before) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다.

-> Jointpoint 앞에서 실행되는 Advice



After Returning 어드바이스

타겟의 메서드가 정상적으로 실행된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다.

-> Jointpoint 메서드 호출이 정상적으로 종료된 뒤에 실행되는 Advice



After Throwing 어드바이스

타겟의 메서드가 예외를 발생된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의한다.

-> 예외가 던져 질때 실행되는 Advice


조인포인트를 어노테이션으로도 줄수 있음

@EnableAspectJAutoProxy
-> 

ProxyTargetClass
->

joinPoint.getArgs() 이걸로 타켓 메소드에 들어가는 parameter 정보를 가져올 수 있다.

pointjoint 할때 쓰는 expression 들도 알아야 할듯

리플렉션하는 소스를 스테이틱이 아닌 스프링 빈을 활용한 싱글톤 형태로 구현하면.
리플렉션시 빈을 주입받지 못하는 오류를 해결할 수 있다.


## 주제. AOP

1. 기본적인 로깅

2. 중복되는 로깅 소스 발견 -> 새로운 로깅 객체 생성 하여 콜

3. 비지니스 로직에 새로운 로깅 객체를 계속 콜하는 소스가 발견 -> 메인 비지니스 소스에는 로깅 같은 소스를 안넣었으면 좋겟음. -> proxy 패턴 사용

4. 인터페이스를 활용한 proxy 패턴 확장

5. Spring AOP 활용.

6. Annotation 활용한 AOP

7. Spring AOP + custom annotation + SpEL



## 주제. Annotation

```java

package reflection;

import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * 어노테이션 정의
 */
@Retention(RUNTIME)
public @interface Check {
    String value();
}
```

```java

package reflection;

/**
 * 조작 대상 클래스
 */
@Check("클래스에 부여")
public class AnnotationSample {

    @Check("메소드에 부여")
    public void print(@Check("인수에 부여") String message) {
        System.out.println(message);
    }
}

```

```java

package reflection;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;

public class AnnotationMainSample {

  public static void main(String[] args) throws NoSuchMethodException {

    Class<AnnotationSample> clazz = AnnotationSample.class;

    /////////////////////////////////////////////////////////////////////////////
    // 클래스에 부여한 @Check를 얻기
    /////////////////////////////////////////////////////////////////////////////
    {
        Check check = clazz.getAnnotation(Check.class);
        System.out.println(check);
        System.out.println(check.value());
    }

    /////////////////////////////////////////////////////////////////////////////
    // 메소드에 부여한 @Check를 얻기
    /////////////////////////////////////////////////////////////////////////////
    Method method = clazz.getMethod("print",  String.class);
    
    // @Check가 부여되어 있는 경우
    if (method.isAnnotationPresent(Check.class)) {
        Check check = method.getAnnotation(Check.class);
        System.out.println(check);
        System.out.println(check.value());
    } 

    /////////////////////////////////////////////////////////////////////////////
    // 인수에 부여한 @Check를 얻기
    /////////////////////////////////////////////////////////////////////////////
    for (Annotation[] params : method.getParameterAnnotations()) {
        for (Annotation annotation : params) {
            Check check = (Check) annotation;
            System.out.println(check);
            System.out.println(check.value());
        }
    }
  }
}

```

```java

import java.lang.annotation.*;

@Inherited
@Documented
@Retention(RetentionPolicy.RUNTIME) // 컴파일 이후에도 JVM에 의해서 참조가 가능합니다.
//@Retention(RetentionPolicy.CLASS) // 컴파일러가 클래스를 참조할 때까지 유효합니다.
//@Retention(RetentionPolicy.SOURCE) // 어노테이션 정보는 컴파일 이후 없어집니다.
@Target({
        ElementType.PACKAGE, // 패키지 선언시
        ElementType.TYPE, // 타입 선언시
        ElementType.CONSTRUCTOR, // 생성자 선언시
        ElementType.FIELD, // 멤버 변수 선언시
        ElementType.METHOD, // 메소드 선언시
        ElementType.ANNOTATION_TYPE, // 어노테이션 타입 선언시
        ElementType.LOCAL_VARIABLE, // 지역 변수 선언시
        ElementType.PARAMETER, // 매개 변수 선언시
        ElementType.TYPE_PARAMETER, // 매개 변수 타입 선언시
        ElementType.TYPE_USE // 타입 사용시
})
public @interface MyAnnotation {
    /* enum 타입을 선언할 수 있습니다. */
    public enum Quality {BAD, GOOD, VERYGOOD}
    /* String은 기본 자료형은 아니지만 사용 가능합니다. */
    String value();
    /* 배열 형태로도 사용할 수 있습니다. */
    int[] values();
    /* enum 형태를 사용하는 방법입니다. */
    Quality quality() default Quality.GOOD;
}

```

위와같이 다양한 형태의 값을 넣을 수 있음


```java

@Around(value = "@annotation(domActionLog)")
    public Object domActionLogging(ProceedingJoinPoint joinPoint, DomActionLog domActionLog) throws Throwable {
        log.info(domActionLog.toString());

        log.info("REQ : " + Arrays.toString(joinPoint.getArgs()));

        Class<AirlineRS> clazz = AirlineRS.class;

        Method method = clazz.getMethod("pnrStatus", PnrStatus.Req.class);
        if(method.isAnnotationPresent(DomActionLog.class)) {
            DomActionLog annotation = method.getAnnotation(DomActionLog.class);
            System.out.println(annotation);
            System.out.println(annotation.prId());
            System.out.println(annotation.pnr1());
        }

        Object obj = null;
        try {
            obj = joinPoint.proceed();
            log.info("RES : " + obj.toString());
        } catch (Exception e) {
            log.info("RES : " + e.toString());
        }

        return obj;
    }

```

DomActionLog 부분을 Around annotation 이름과 맞추면 파라미터로 받을 수 있다.




내가 만들어둔 소스

dependencies

```
	compile group: 'org.aspectj', name: 'aspectjrt', version: '1.9.2'
	compile group: 'org.aspectj', name: 'aspectjweaver', version: '1.9.2'
```

config (It should be turned on @EnableAspectJAutoProxy with proxyTargetClass = true option)

```java

@Configuration
@EnableWebMvc // <annotation-driven />
@EnableAspectJAutoProxy(proxyTargetClass = true)
@ComponentScan(basePackages = { "com.air.*" })
@Slf4j
public class WebConfig extends WebMvcConfigurerAdapter{
    ...
}

```

annotation

```java

import java.lang.annotation.*;

@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DomActionLog {
    String prId();
    String pnr1();
    String param() default "";
}


```

Aspect Source

```java

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;



@Slf4j
@Aspect
@Component
public class CommonAspect {

    @Around(value = "@annotation(DomActionLog)")
    public Object domActionLogging(ProceedingJoinPoint joinPoint) throws Throwable {
        
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        DomActionLog annotation = method.getAnnotation(DomActionLog.class);

        Object pnr1 = getDynamicValue(signature.getParameterNames(), joinPoint.getArgs(), annotation.pnr1());
        Object prId = getDynamicValue(signature.getParameterNames(), joinPoint.getArgs(), annotation.prId());

        Object result = joinPoint.proceed();

        return result;
    }

    public static Object getDynamicValue(String[] parameterNames, Object[] args, String key) {
        ExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();

        for(int i$=0; i$< parameterNames.length; i$++) {
            context.setVariable(parameterNames[i$], args[i$]);
        }

        return parser.parseExpression(key).getValue(context, Object.class);
    }
}


```

target source

```java

    @Override
    @DomActionLog(prId = "#req.prId", pnr1 = "#req.pnr1")
    public PnrStatus.Res pnrStatus(PnrStatus.Req req) throws Exception {
        /* 필수 파라미터 확인 */
        if(req == null || StringUtils.isEmpty(req.getPnr1())) {
            throw new Exception("파라미터 'pnr1' 정보가 null 이거나 공백입니다");
        }

        /* 리트리브 */
        RetrieveDetail retrieveDetail = (RetrieveDetail) jaxb.unMarshall(RetrieveDetail.class,
                airLineGateway.call(
                        AirLineGateway.ServiceUrl.RETRIEVE.name(),
                        airLineRQBuilder.retrieveRQBuilder(new PassengerRecordVO(req.getPnr1(), "-1", req.getAirline()), ""),
                        req.getAirline()
                )
        );

        /* 취소 리트리브 */
        List<RetrieveResult> retrieveResults = new ArrayList<>();
        for(String ticketNo : this.ticketNos(retrieveDetail)) {
            retrieveResults.add((RetrieveResult) jaxb.unMarshall(RetrieveResult.class,
                    airLineGateway.call(
                            AirLineGateway.ServiceUrl.TICKET_RETRIEVE.name(),
                            airLineRQBuilder.ticketDetailRetrieve(req.getPnr1(), "", req.getAirline(), Collections.singletonList(ticketNo)),
                            req.getAirline()
                    )
            ));
        }

        return this.pnrStatus(retrieveResults);
    }

```

Test code 

```java

    @Test
    public void run() throws Exception {
        PnrStatus.Res res = airlineRS.pnrStatus(new PnrStatus.Req.Builder().airline("RS").prId("50001234").pnr1("JR85F").build());
    }

```