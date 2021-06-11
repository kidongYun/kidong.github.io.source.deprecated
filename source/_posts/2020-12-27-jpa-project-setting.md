---
layout: post
title:  "JPA 프로젝트 기본 설정에 대한 고찰"
date:   2020-12-27 11:38:54 +0900
categories: spring
---

이 글은 백기선 개발자님의 Spring Data JPA 강의를 듣고 JPA 관련 내용들을 정리한 글이다. 기본적으로 해당 강좌에서 언급되는 개념으로 진행되지만, 저자의 JPA 경험을 토대로 이해한 바가 함께 녹아있으므로 그 부분은 참고하고 글을 읽어주시기 바란다.

### JPA 프로그래밍 프로젝트 세팅

Maven 프로젝트를 활용해서 간단하게 JPA 프로그래밍이 가능한 환경을 구축해보자. 우선 의존성을 추가한다. JPA 관련 기능이 담겨져 있는 'spring-boot-starter-data-jpa' 라이브러리와 당신이 사용할 데이터베이스의 JDBC 드라이버를 가져오자 여기서는 postgresql 데이터베이스를 사용하기 때문에 이 의존성을 가져왔다.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.whiteship</groupId>
    <artifactId>demospringdata</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demospringdata</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

postgresql 데이터베이스에 접속하기 위해 필요한 정보 url, username, password 를 입력하자. 그리고 개발의 편의를 위해 추가적인 설정들도 넣었는데 이는 주석으로 내용을 설명해 두었다. 지금과 같이 application.properties에 설정한 정보들은 HibernameJpaAutoConfiguration 이 자동 설정을 해준다.

```

spring.datasource.url=jdbc:postgresql://localhost:5432/springdata
spring.datasource.username=keesun
spring.datasource.password=pass

// 스프링 서버 재시작시 DDL을 항상 새로만들지, 업데이트할지, 검증만할지를 결정한다.
spring.jpa.hibernate.ddl-auto=create

spring.jpa.properties.hibername.jdbc.lob.non_contextual_creation=true

// 우리가 만든 JPA 쿼리가 기존 SQL 쿼리로 어떻게 변환되어 날라가는지 보여준다.
spring.jpa.show-sql=true
// 이 쿼리를 개행 등을 하여 좀 더 보기 쉽게 보여준다
spring.jpa.properties.hibernate.format_sql=true

```

spring.jpa.hibernate.ddl-auto=create 이 속성은 DDL을 서버 재시작 할 때마다 새로 생성하기 때문에 개발할 때 유용하며,
실제 운영 단계에서는 'validate' 값을 주는 게 좋다. 'update' 값은 기존 스키마와 데이터들을 유지하고 새로운 값이 들어오면 생성한다. 'update' 값의 주의사항은 기존 값들을 유지하기 때문에 편하긴 하지만 무조건 좋은건 아니다. 왜냐하면 특정 컬럼을 지우는 DDL을 'update' 값으로 날렸을 경우 기존 스키마를 유지해야 하기 때문에 지우고 싶었던 필드가 실제로 릴레이션에서 보면 남아있다. 또 'update' 값으로 기존 스키마의 이름을 바꾸면 변경이 안된다. 단지 기존 것을 남기고 새로운 걸 추가한다. 이렇게 이전에 작업하던 것들이 지속 남아있게 됨으로 'update' 값을 사용하면 스키마가 지저분해질 수 있으므로 주의해서 사용해야 한다.

그 다음은 도메인 객체를 생성한다. 이 객체가 자바에서 사용하는 도메인 객체이면서 데이터베이스의 릴레이션 즉 테이블이 된다.

```java

@Getter
@Setter
@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;
}

```

lombok 어노테이션은 제외하고 JPA와 관련된 어노테이션에 대해서만 간략히 알아보도록 하자.

```

@Entity : JPA에서 사용하는 도메인 객체임을 선언하는 어노테이션이며, 선언된 객체는 릴레이션으로 매핑된다.

@Id : 해당 필드가 PK 값으로 쓰인다고 명시한다.

@GeneratedValue : mysql에서 auto_increment 같은 것으로 생각하면 되며 값을 자동으로 생성시켜준다. 생성 방식은 옵션으로 추가할 수 있으며 데이터베이스에 따라 SEQUENCE를 생성할지, Identity 테이블을 사용할지를 결정한다.

@Column : 위 예제 코드에는 이 어노테이션이 안보이지만 @Entity 어노테이션을 사용한 도메인 객체의 필드들은 이 어노테이션이 자동으로 붙는다.
이 어노테이션을 통해서 실제 릴레이션에 매핑되는 컬럼의 이름등을 변경할 수 있다.

```

빙금 만든 Account 도메인 객체가 릴레이션에 잘 매핑되고 있는지 확인하기 위해서 Runner 클래스 하나를 생성하자.

```java

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.transaction.Transactional;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("keesun");
        account.setPassword("jpa");

        entityManager.persist(account);
    }
}

```

Spring Framework에서 핵심 객체라고 하면 ApplicationContext가 있듯이 JPA에서는 EntityManager가 가장 핵심 객체라고 할 수 있다. 이 객체를 활용하여 영속화가 가능하다. 즉 데이터베이스에 실제 쿼리를 날릴 수 있다. JPA에서 제공하는 이 EntityManger 객체를 살펴보면 하이버네이트 기반으로 구현되어 있는 Session 객체에 접근할 수 있다. 영속화를 위해서 이 두 객체 모두 사용이 가능하다. 그렇기 때문에 JPA를 사용한다고 하면 하이버네이트 기반의 코딩도 실제로는 가능하다. 아래 예제는 하이버네이트 방식으로 영속화를 하는 코드이다.

추가적으로 이 Runner 클래스는 @Transactional 어노테이션을 사용하고 있는데 이 어노테이션은 JPA에서 제공하는 어노테이션이 아니다. Entity와 관련된 작업은 트랜잭션 범주를 설정하고 그 안에서 쿼리들이 이뤄져야 한다. 대표적인 예시로 은행 ATM 입/출금 기능을 들 수 있겠다.
나의 계좌에서 만원을 출금하는 경우 출금하는 돈 항목에는 만원이 추가되고, 나의 계좌에는 만원이 빠져나가야 한다. 그러나 만약 출금하는 돈에 만원이 추가된 이후에 오류가 나서 나의 계좌에서 만원이 빠져나가지 않는 경우 고객은 만원을 벌게되는 치명적 오류가 발생한다. (나에게 이런 오류가 발생했으면 좋겠다..) 이렇듯 데이터베이스 관련 업무는 한 트랜잭션의 범주를 설정하고 한 세트로 동작하도록 하는 것이 중요한데 이를 위해 @Transactional 어노테이션을 붙인다.

@Transactional 어노테이션은 클래스 기준으로 사용하게되면 해당 클래스가 가지고 있는 모든 메서드에 트랜잭션 관리가 적용이 되고 한 메서드에만 적용하면 해당 메서드에만 적용이 된다.

```java

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.transaction.Transactional;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("keesun");
        account.setPassword("hibernate");

        /* JPA 객체인 EntityManager에서 하이버네이트 객체인 Session을 꺼내온다 */
        Session session = entityManager.unwrap(Session.class);
        session.save(account);
    }
}

```