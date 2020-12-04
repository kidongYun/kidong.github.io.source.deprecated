---
layout: post
title:  "Spring AOP + Custom Annotation + SpEL"
date:   2020-12-04 11:38:54 +0900
categories: spring
---

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