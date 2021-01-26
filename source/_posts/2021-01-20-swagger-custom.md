---
layout: post
title:  "Swagger 커스터마이징"
date:   2021-01-20 11:38:54 +0900
categories: spring
---

### Swagger 에 Authentication Header 추가

```java

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.ant("/api/**"))
                .build()
                .securityContexts(Collections.singletonList(securityContext()))
                .securitySchemes(Collections.singletonList(apiKey()));
    }

    private ApiKey apiKey() {
        return new ApiKey("JWT", "x-auth-token", "header");
    }

    private SecurityContext securityContext() {
        return springfox
                .documentation
                .spi.service
                .contexts
                .SecurityContext
                .builder()
                .securityReferences(defaultAuth()).forPaths(PathSelectors.any()).build();
    }

    List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        return Collections.singletonList(new SecurityReference("JWT", authorizationScopes));
    }
}

```

### Swagger Model Naming 정책 변경하기.

기본적으로 Swagger 는 리플렉션시 getSimpleName() 함수를 사용. 이렇게되면 Nested Class 가 만약 이름이 동일하다면 Swagger-ui 에서 모델을 구분하지 못한다. 이럴때 네이밍 정책을 바꿔준다.

```java

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.ant("/api/**"))
                .build()
                .securityContexts(Collections.singletonList(securityContext()))
                .securitySchemes(Collections.singletonList(apiKey()));
    }

    private ApiKey apiKey() {
        return new ApiKey("JWT", "x-auth-token", "header");
    }

    private SecurityContext securityContext() {
        return springfox
                .documentation
                .spi.service
                .contexts
                .SecurityContext
                .builder()
                .securityReferences(defaultAuth()).forPaths(PathSelectors.any()).build();
    }

    List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        return Collections.singletonList(new SecurityReference("JWT", authorizationScopes));
    }

    @Component
    @Order(SwaggerPluginSupport.SWAGGER_PLUGIN_ORDER)
    public static class CustomTypeNameProvider implements TypeNameProviderPlugin {
        @Override
        public String nameFor(Class<?> type) {
            String fullName = type.getName();
            return fullName.substring(fullName.lastIndexOf(".") + 1);
        }

        @Override
        public boolean supports(DocumentationType documentationType) {
            return true;
        }
    }
}

```