---
layout: post
title:  "SPRING DATA JPA"
date:   2020-12-24 11:38:54 +0900
categories: spring
---

객체와 DB 사이에 매핑을 해주는 것은 ORM 에서 굉장히 일부의 가치이다.

ORM의 구현체인 JPA, 하이버네이트.


### 관계형 데이터베이스와 자바

테이블과, 연관관계를 활용해서 데이터를 식별하고 관리하는 그런 것들이 관계형 데이터베이스

JDBC를 활용해 데이터베이스에 접속하고 사용하는 것.

영속화 -> 데이터를 영속화할 필요성이 있다.
즉 어딘가에 저장을 해놓고 우리 어플리케이션을 꺼따 켜도 이 데이터들이 유지가 되어야한다.

영속화의 방법은 다양하다 -> 파일에다가 저장해도 되지만 여기서는 관계형 데이터베이스에 데이터를 넣고 여기에 조회하는 방법을 살펴보는 것.

먼저 postgres DB 를 도커 이미지로 다운받아서 돌리고 postgresql 설정을 진행하자

```
> docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springdata --name postgres_boot -d postgres

> docker exec -i -t postgres_boot bash

> psql --username keesun --dbname springdata

```

postgresql 관련 JDBC 의존성 주입 

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>me.whiteship</groupId>
    <artifactId>jdbcsample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.2</version>
        </dependency>
    </dependencies>

</project>

```

JDBC 연동 소스 작성

```java

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Application {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/springdata";
        String username = "keesun";
        String password = "pass";

        try(Connection connection = DriverManager.getConnection(url, username, password)) {
            System.out.println("Connection created: " + connection);
        }
    }
}

```

JDBC
    DataSource / DriverManager
    Connection
    PreparedStatement

SQL
    DDL -> 스키마 생성
    DML -> SELECT, UPDATE, INSERT, DELETE


try with resource 문법의 장점
자원에 연동하고 그다음에 연결이 끊어져야하는 이런 소스들에 대해서 자원의 해지를 다 사용하고 난뒤에 알아서 처리를 해준다.

테이블 생성

```java

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Application {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/springdata";
        String username = "keesun";
        String password = "pass";

        try(Connection connection = DriverManager.getConnection(url, username, password)) {
            System.out.println("Connection created: " + connection);
            String sql = "CREATE TABLE ACCOUNT (id int, username varchar(255), password varchar(255));";

            try(PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.execute();
            }
        }
    }
}


```

데이터 삽입

```java

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Application {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/springdata";
        String username = "keesun";
        String password = "pass";

        try(Connection connection = DriverManager.getConnection(url, username, password)) {
            System.out.println("Connection created: " + connection);
            String sql = "INSERT INTO ACCOUNT VALUES(1, 'keesun', 'pass');";

            try(PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.execute();
            }
        }
    }
}

```

무엇이 문제일까??

1. 테이블에 있는 데이터를 가져와서 자바 도메인 객체로 바꿔야 하는 작업이 번거롭다.
2. 커넥션을 만드는 작업이 굉장히 비싼 작업 (오래거리는 일) -> 그래서 보통 pool로 관리 (DBCP) 스프링 부트는 히카리 커넥션 이거 내꺼 프로젝트에도 해놈
3. SQL이 어느정도는 표준이지만 DB마다 다르다. 그래서 DB 종류를 바꾸면 쿼리도 바껴야한다.
4. 중복되는 작업이 많다 -> 쿼리만 다를 뿐 다른 연결하는 작업들은 모두 중복되는 작업
5. 모든데이터를 가져오는게 아니고 필요한 데이터만 가져올떄가 있음 필요할때 데이터를 가져오는것 (Lazy loading) 쿼리로도 할수 있지만 이를 모두 구현하기가 쉽지는 않다.


### ORM 개요

Object-Relation Mapping

Object : 도메인 모델
Relation : DB의 테이블
둘을 매핑한다.

도메인 모델을 사용해서 DB 작업을 할 수 있는게 왜 좋을까?

1. 자바에서 비지니스 로직을 구현하려면 결국 SQL로 데이터를 가져와도 결국 도메인 모델로 매핑해야함.
2. 디자인패턴도 사용이 가능하다
3. 우리의 비지니스 로직에 집중이 가능해진다.
4. 코드의 재사용성이 늘어난다 (의존성이 줄어든다)

ORM은 애플리케이션의 클래스와 SQL 데이터베이스의 테이블 사이의 매핑 정보를 기술한 메타데이터를 사용하여, 자바 애플리케이션의 객체를 SQL 데이터베이스의 테이블에 자동으로 또 꺠끗하게 영속화 해주는 기술입니다.

DDD의 의미가 이런건가 JPA를 쓰는것 같은??

JDBC의 연동을 위해 만들어야하는 그런 소스 없이 도메인 만으로 작성.
(비 침투적이다.)

비침투적이라는 것은 비지니스 개발자가 이 JPA 라이브러리를 쓰는데 라이브러리를 위한 코드를 쓰지 않는다는 건데 완전히 비침투적이진 않다.
EntityManager 등을 쓰기때문에.

Spring Framework 도 동일하게 비침투성을 가진다.

하이버네이트를 사용하면 1. 생산성
도메인 데이터를 왔다갔다하는게 굉장히 쉽다.

2. 유지보수성
코드가 비지니스만 남고 그렇기때문에 짧아지고 테스트코드 짜기도 쉽고 가독성이 높아진다.

3. 성능
단건 : JPA 가 SQL 보다 느릴 수 있다.
이 얘기는 C와 자바 중에 누가 성능이 좋은가 이런 얘기임
JPA를 잘쓰면 성능을 좋게끔 만들수도 있다는 의미.

JPA는 객체와 DB 사이에 캐시가 있어서 변경사항들을 고려하고 반영해야하는지 아닌지를 결정함. 이 캐싱을 잘 활용하면 성능이 좋아질 수도 있는거임.

4. 벤더 독립성.
벤더 = 데이터베이스 종류를 말하는듯.
데이터베이스의 종류에 상관없이 ORM은 동일한 형태로 짤수 있다.
JVM의 개념과 사실 동일하지 않나 라고 생각.

단점

1. 학습비용
SQL 도 알아야함. 즉 둘다 알아야 ORM을 잘 쓸수 있음.
배워야할게 엄청 많나봄. 성능 튜닝까지 하려면 개 많이 알아야 하나보다.

즉 하이버네이트는 배우는데 오래걸리는 프레임 워크 중 하나. 
어떻게 보면 스프링프레임워크보다도 더 러닝커브가 높다고도 봄.

하이버네이트를 많이 학습하면 할수록 성능이슈를 높일 수 있다.

하이버네이트 밑단에서는 JDBC 에 있다. 즉 그말은 커스터마이징도 가능하다. 하이버네이트로 부족하다면 더 로우하게 학습하고 커스터마이징 하면 성능 올리수 있다.

### ORM: 패러다임 불일치

객체와 릴레이션사이에 매핑을 할떄 서로 잘 맞지 않는 부분.

1. 밀도의 문제

```java

class Account {
    Long id;

    Address address;

    List<Study> studies; 
}

class Account {
    Long id;

    String address;

    List<Study> studies;
}

```

 객체는 커스텀한 타입을 만들어내기가 쉽지만 릴레이션은 그렇지 않다. 테이블 타입, 기본 데이터 타입이 전부. 이 두 타입 간의 매칭 불일치 문제. ex)날짜 타입

 2. 서브타입 (상속)

 객체는 클래스간의 상속구조를 만들기가 쉽다. 그게 사실 기본인데 릴레이션에는 상속이라는 개념이 없다.
 이를 표현하는 표준적인 기술은 없다.
 
 ORM에서 이에대한 대안은 여러가지로 제안해줌.

3. 데이터 네비게이션 문제.

요건 먼소리냥.

4. 관계 문제

```java

public class User {
    List<Study> myStudy;
}

public class Study {
    List<User>
}

```

위와 같이 객체는 필드를 가지고 어떤 객체를 참조하고 있는지 그 관계를 표현할 수 있다.

또 위처럼 User가 Study를 참조하고 있는 것처럼 그 방향성에 대한 부분도 위 객체 생성시 명시가 가능하다.

또 다대다 관계를 가질 수 있다.

릴레이션은 외래키로 관계를 표현하며 방향성이 사실 없다.

태생적으로 다대다 관계를 만들지 못하면 조인테이블, 링크 테이블을 활용해서 묶어야함.

이런 관계를 표현하는 방법에 미매칭.

Study 테이블에서 User 테이블의 FK 를 가지고 있다고 하자. 두 테이블을 조인하고 이 FK 값을 비교해서 결국 두 테이블의 원하는 방향성으로 데이터를 가져올 수 있따.
즉 방향성은 양방향임.

릴레이션은 다대다가 없기 때문에 ManyToMany 어노테이션을 사용하면 JPA, 하이버네이트가 내부적으로 이를 조인테이블을 만들어서 작업해줌.

5. 데이터 네비게이션 문제.

객체 내부에서의 네비게이션은 각 객체.필드 형태로 계속 순회가 가능함. 다 돌아다닐 수 있음.

릴레이션은 이렇게 하려면 가능은하지만 한번의 조회가 다 쿼리를 날리는 거기때문에 성능이 안좋다.
sql은 커넥션 생성 작업이 비싸기때문에 성능을 올리려면 한번의 쿼리로 날리는게 좋다.

그러나 한번의 쿼리로 날린다는 것은 모두 조인한다는 건데. 이 많은 데이터를 한번에 가져오는게 또 좋은것인가.
모두다 쓰는데이터가 맞는가.

이 둘을 잘 조율 해야하나비ㅏ.

자주접근하는것도 안좋고 많이 접근하는것도 안좋다.

->  lazy loading 을 하자. -> n +1  문제 발생

객체와 릴레이션의 패러다임이 다르기 때문에 이 갭을 채우기위한 노력을 해야함.


### JPA 프로그래밍 프로젝트 세팅

의존성 추가

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

EntityManager 가 내부적으로 하이버네이트를 사용함 그래서 JPA, 하이버네이트를 직접 사용할 수는 있지만.
여기서는 Spring Data JPA를 사용할 것임

application.properties 에 기본설정 application.properties 정보는 HibernameJpaAutoConfiguration 이 자동 설정을 해줌.

```

spring.datasource.url=jdbc:postgresql://localhost:5432/springdata
spring.datasource.username=keesun
spring.datasource.password=pass

spring.jpa.hibernate.ddl-auto=create
spring.jpa.properties.hibername.jdbc.lob.non_contextual_creation=true

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

```

spring.jpa.hibernate.ddl-auto=create 이 속성은 개발할 때 유용하며,
실제 운영 단계에서는 validate 값을 주는 게 좋다. update는 기존 스키마를 유지하고 새로운 것들만 생성한다. update 의 주의사항은 새로운 컬럼을 추가할 때 자동으로 추가해주는게 편하긴 한데 좋은건 아니다. 왜냐면 반대로 기존 컬럼을 지우려고할때 도메인 객체에서는 지웠지만 실제로 릴레이션에서 보면 남아있다. update를 사용하면 스키마가 지저분해질 수 있따. 또 update를 하면 기존 스키마의 이름을 바꾸면 변경이 안되고 그저 기존 것을 남기고 새로운 걸 추가한다.

```java

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

@Entity -> 도메인 객체이며 릴레이션과 매핑하겠다라고 선언
@Id -> PK 항목임을 선언
@GeneratedValue -> 값을 자동으로 생성해서 사용하겠다라고 선언 (mysql 의 AUTO_INCREMENT 같은)
username, password 같은 필드들도 위에서 @Entity 선언을 했기때문에 저 이름으로 컬럼이 생성됨 = @Column 이라는 어노테이션이 생략된것임.

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

EntityManager가 JPA에 가장 핵심 객체 이를 활용하여 영속화가 가능하다. Entity와 관련된 모든 작업들은 한 트랜잭션 안에서 이루어져야함.
은행 ATM 출금 기능이 예가 될수 있겠다. DB 업무는 트랜잭션 처리가 정말 중요한듯. 여하튼 이를 위해 @Transactional 어노테이션을 붙여준다.

클래스에 @Transactional 를 붙이면 해당 클래스가 가지고 있는 모든 메서드에 적용이 되고 메서트에 적용하면 해당 메서드에만 적용이 된다.

JPA가 HIBERNAME를 사용하기 때문에 실질적으로 둘 다 사용이 가능하다. HIBERNAME의 가장 핵심적인 메서드는 Session 이다 아래는 Hibernate를 활용해서 영속성 관리를 하는 코드이다. entitymanager 하단에 있는 session 객체를 꺼내오고, 그걸로 save 하고있다.

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

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
    }
}

```

### JPA 프로그래밍 2. 엔티티 타입 매핑

어노테이션 방법과 xml 방법 두가지가 있음 그런데 요즘은 거의 어노테이션 방법만 사용.

@Entity 어노테이션 안에는 @Table 어노테이션이 생략 되어있음. @Entity 어노테이션에 이름을 줄수 있따. 안주면 클래스 이름을 기본적으로 사용,
주면 준 이름을 사용 이게 테이블까지 가지는 않는다 @Entity 어노테이션에 설정한 이름은 객체 세상에서만 사용되어지는 거고 테이블의 이름을 바꾸고 싶으면 @Table 어노테이션을 사용해야한다. @Table 어노테이션의 기본 이름 설정은 @Entity 어노테이션의 이름이기 때문에 테이블의 이름이 바뀌는것이였다.

@Id 는 주키를 설정하는 어노테이션. 문서상에는 모든 타입을 제공한다. 여기서 논쟁거리 하나가 primitive type을 쓸것이냐 혹은 reference type을 쓸것이냐.

```java

/* primitive type */
@Id
private long id;

/* reference type */
@Id
private Long id;

```

장점은 long type은 아무값도 안 넣었을때에 0이다. 근데 이 0이 어떤걸 의미할 수도 있다 (의도하지 않았지만) Long은 null로 들어가기때문에 이에대한 처리가 가능. reference 타입을 쓰는 장점이 이거구나..

@GeneratedValue 어노테이션은 자동생성된 값을 사용하겠다는 어노테이션. 기본적으로는 각 데이터베이스의 룰에 맞춰서 생성이됨 SEQ를 쓸건지, Identity 객체를 쓸건지.. 등등 그러나 명시적으로 사용할수 있따. @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = '')  이것 처럼

@Column 컬럼에 사용하는 어노테이션 -> 추가적인 옵션 넣는거가 일반적으로
@Column(nullable = false, unique = true) 요런식. 

@Temporal 요거는 날짜시간 관련해서 매핑이 가능하다. 3가지 속성이 있다 그떄그떄 맞춰서 쓰자 LocalDate는 자바 1.8 이후에 들온거라서 JPA 2.1 전에는 이 타입에 대한 매핑이 지원이 안된다. Calendar와 Date만 된다. 할수는 있으나 하이버네이트를 커스텀 타입으로 쓰는 고급 기술을 써야한다고 함.

@Transient 어노테이션을 사용하면 이 필드는 컬럼으로 매핑을 안하겟다는 의미이다.

spring.jpa.show-sql=true -> JPA가 자동으로 처리하는 쿼리 내용들을 보여달라는 설정
spring.jpa.properties.hibernate.format_sql=true -> 저 쿼리 내용들을 보기 쉽게 보여달라는 설정 (단순히 beautify 기능)

### JPA 프로그래밍 3. Value 타입 매핑

Value 타입은 필드를 이야기하는 것 같음. 엔티티 타입은 클래스이고. 여기서는 Reference 타입의 Value에 대한 매핑 하는 방법을 알려주려는 것 같다.
대표적인 예로 Address 필드.

엔티티로써 사용할만큼 큰 커스텀 클래스라면 @Entity 어노테이션을 써서 기존처럼 새로운 릴레이션을 만들면 된다. 그러나 지금처럼 Address의 경우 그정도의 사이즈는 아니고 단순하게 값을 묶어서 가지고있는 정도이기 때문에 이녀석을 @Embeddable, @Embedded 어노테이션을 활용하면 객체에서는 묶여있지만 릴레이션에서는 모두 포함된 구조로 구현이 가능하다.

이런 타입은 @Embeddable 을 활용해서 만들고, 넣으려고하는 엔티티 객체에 @Embedded 어노테이션과 함꼐 제공하여야 한다.
```java

@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
}

```

같은 Embeddable 객체를 한 엔티티 내에 여러번 사용할 경우에는 @AttributeOverrides, @AttributeOverride 어노테이션을 활용해 이름을 오버라이딩할 수 있따.

### JPA 프로그래밍 4. 관계 매핑

1:N 관계 매핑 합시다! 두 개의 엔티티가 서로 맞물려야 할때 이 관계 매핑을 해야한다. 내 객체를 여러 명이 참조가 가능하다. 
이렇게 하면 실제 릴레이션에는 Account 릴레이션의 PK를 참조하는 FK 값을 Study 릴레이션에 생성해준다. 이 관계에서 방향성은 Study가 Account를 의존하고 있기때문에 참조하고있기때문에 현재 Study가 주인인 객체이다.

```java

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account ownwer;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Account getOwnwer() {
        return ownwer;
    }

    public void setOwnwer(Account ownwer) {
        this.ownwer = ownwer;
    }
}

```

이번엔 Account가 주인이라고 생각해보자 Account가 한명이 여러개의 Study를 가질 수 있으므로 아래와 같이 생성하고 @OneToMany 어노테이션을 붙인다.
@OneToMany 어노테이션이 릴레이션에 매핑될 때에는 기본적으로 조인테이블이 생성된다.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany
    private Set<Study> studies = new HashSet<>();

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Set<Study> getStudies() {
        return studies;
    }

    public void setStudies(Set<Study> studies) {
        this.studies = studies;
    }
}


import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

양방향 참조를 하려면? Study 객체에 Account 객체를 참조하는 필드를 하나 생성하고 @ManyToOne 어노테이션을 붙인다. 이렇게하면 단뱡향으로 2개의 연결관계가 생긴다. 이걸 양방향으로 하고싶다면. Account 객체에서 @OneToMany 속성중 mappedBy에 Study 객체가 Account 객체를 참조하기 위해 사용한 필드의 이름을 넣어준다.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Set<Study> getStudies() {
        return studies;
    }

    public void setStudies(Set<Study> studies) {
        this.studies = studies;
    }
}

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account owner;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

그리고 실제로 데이터를 넣을때에는 결국 도메인 레벨에서는 객체지향 프로그래밍이기 때문에 양 객체에 모두 참조를 넣어주어야 한다.

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
        account.setUsername("kidongyun");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.getStudies().add(study);
        study.setOwner(account);

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
        session.save(study);
    }
}

```

이렇게 한세트로 다니기 때문에 보통 주인인 객체에 함수로 만들고 이들을 함께 호출함.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Set<Study> getStudies() {
        return studies;
    }

    public void setStudies(Set<Study> studies) {
        this.studies = studies;
    }

    public void addStudy(Study study) {
        this.getStudies().add(study);
        study.setOwner(this);
    }

    public void removeStudy(Study study) {
        this.getStudies().remove(study);
        study.setOwner(null);
    }
}

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
        account.setUsername("kidongyun");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.addStudy(study);

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
        session.save(study);
    }
}

```

### JPA 프로그래밍 5. 엔티티 상태와 Cascade

Cascade 속성은 @ManyToOne, @OneToMany 어노테이션 옵션으로 가지고 있는데. 의미는 해당 엔티티 상태의 변화를 전파하겠다라는 의미이다.
이는 DB에서 cascade 옵션이랑 유사한것 같음.

```java

@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account owner;
}

```

위 코드에서 Study 엔티티가 A 상태에서 B 상태로 전이될때 Account 객체에도 이 전이를 전파하고 싶을때 위와 같이 cascade 옵션으로 제공하면 된다. 기본값은 전파를 하지 않음.

엔티티의 상태는 총 4가지가 있음.

1. Transient
```java

public void run() {
    Account account = new Account();
    account.setUserName("keesun");
    account.setPassword("jpa");
}

```

객체 수준에서는 account 객체를 만든것을 인지하고 있지만 이것이 하이버네이트까지 전달되지 않아 릴레이션에는 없는 상태를 Transient 라고합니다.

2. Persistent

```java

session.save(account);

```

위에서 만든 account 인스턴스를 하이버네이트 세션을 통해 영속화를 한 이후에는 Persistent 상태로 바뀐다.

save를 했다고 해서 바로 디비에 들어가는 것은 아니고 하이버네이트가 가지고 있다가. 디비에 넣어야겠다고 판단하는 시점에 들어간다.

여하튼 상태는 바뀜. 하이버네이트가 관리하는 객체가 되는 Persistent 상태가 됨.

이때 제공되는 기능들이 있는데 1차캐시. Persistent Context 라는 곳에 캐싱이 됨.

```java

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("kidongyun");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.addStudy(study);

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
        session.save(study);

        Account keesun = session.load(Account.class, account.getId());
        System.out.println("============================");
        System.out.println(keesun.getUsername());
    }
}
```

이런식으로 코딩하게 되면 저 데이터를 디비에서 가져오는게 아니고 1차 캐싱된 Persistent Context 에서 가져옴. 성능의 상승이 있을 수 있다. 위 처럼 코딩하게 되면 keesun 지역변수에 데이터를 가져올때 데이터베이스에서 가져오지 않고 1차 캐시에 해당하는 Persistent Context 영역에서 데이터를 가져온다. 이렇게 캐싱처리를 하고 나서 한 트랜잭션 작업 마무리가 되면 그 이후에 save() 함수를 호출하여 생성한 INSERT 쿼리는 트랜잭션이 끝나고 난 다음에 실제 디비에 반영이 된다.

Dirty checking

```java

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("kidongyun");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.addStudy(study);

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
        session.save(study);

        Account keesun = session.load(Account.class, account.getId());
        keesun.setUsername("whiteship");
        System.out.println("============================");
        System.out.println(keesun.getUsername());
    }
}

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("kidongyun");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.addStudy(study);

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
        session.save(study);

        Account keesun = session.load(Account.class, account.getId());
        keesun.setUsername("whiteship");
        keesun.setUsername("helloworld");
        keesun.setUsername("kidongyun");
        System.out.println("============================");
        System.out.println(keesun.getUsername());
    }
}

```
 
 위의 코드는 keesun.setUsername("whiteship") 이 부분만 추가한 것인데. 요 코드드 사실 상 쿼리와는 연관이 없음에도 하이버네이트가 알아서 인지하고 업데이트 쿼리르 날려줬다. 무슨 뜻이냐면 엔티티 상태가 Persistent 이면 해당 객체를 하이버네이트나 JPA 가 지속 관리를 하고 있다는 것이다. 한 트랜잭션 스코프가 끝났을 때 해당 객체 값이 변경이 이루어 졌다면 변경을 시켜주고, 새로 생성이된 객체가 있다면 INSERT 쿼리를 날려준다. 중요한 사항은 만약 특정 값이 변경이 100번 이루어 졌고 마지막에 결국 초기의 값과 같아졌다면 이 변경 내역 쿼리는 날라가지 않는다. 이는 쿼리 수정내용을 Persistent Context 검증하는 절차를 가진다는 의미이다.

  Dirty Checking 이라는 것은 이렇게 트랜잭션이 끝날떄마다 변경내용을 지속확인하는 것을 의미.
  Write Behind 라는것은 Lazy 기법이랑 비슷한 개념인 것 같다. 마지막에 수정한다는 의미.

3. Detached

Session 이 종료가 되면 Persistent 상태에서 Detached 상태로 넘어간다. 이 상태가 되면  Persistent 상태에서 관리되어지는 다양한 기능들은 동작하지 않는다.
보통 한 Repository 에서 한 트랜잭션이 끝나면 Detached 상태로 전이된다.

```java

/* Detached 상태로 가는 함수들 */
Session.evict();
Session.clear();
Session.close();

/* Re-attached 해사 다시 Persistent 상태가 되는 함수들 */
Session.update();
Session.merge();
Session.saveOrUpdate();

```

4. Removed

------------

Cascading 은 엔티티가 Parent 와 Child 관계인 경우에 주로 사용된다. 부모의 것이 삭제가 되면 자식의 것들도 연쇄적으로 삭제가 되어야 한다.

* Convenience method = 양방향으로 두 객체가 서로 참조하고 있는 경우에 어떤 행위가 일어나면 그거에 대한 작업을 두 방향 모두 해주도록 하는 함수. 내가 직접 만들어야함 위에서 addStudy(), removeStudy() 함수가 이러한 메서드 이다.

Cascading 사용 예시를 위해 부모, 자식 구조를 가지는 Entity Post, Comment 를 구현하자.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
public class Post {
    @Id
    @GeneratedValue
    private Long id;

    private String title;

    @OneToMany(mappedBy = "post", cascade = CascadeType.PERSIST)
    private Set<Comment> comments = new HashSet<>();

    public void addComment(Comment comment) {
        this.getComments().add(comment);
        comment.setPost(this);
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Set<Comment> getComments() {
        return comments;
    }

    public void setComments(Set<Comment> comments) {
        this.comments = comments;
    }
}

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
public class Comment {
    @Id
    @GeneratedValue
    private Long id;

    private String comment;

    @ManyToOne
    private Post post;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public Post getPost() {
        return post;
    }

    public void setPost(Post post) {
        this.post = post;
    }
}

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setTitle("Spring Data JPA 언제 보나...");

        Comment comment = new Comment();
        comment.setComment("빨리 보고 싶어요.");
        post.addComment(comment);

        Comment comment1 = new Comment();
        comment1.setComment("곧 보여드릴게요.");
        post.addComment(comment1);

        Session session = entityManager.unwrap(Session.class);
        session.save(post);

        Post post = session.get(Post.class, 1L);
        session.delete(post);
    }
}


```

위처럼 돌리면 comment를 save 하는 함수가 없지만. post가 자기가 하위로 가지고있는 comment 엔티티가 있고. 이를 Cascading 시켰기 때문에. post save 시에 comment에 관련된 내용이. 전파되어 추가된다. delete 되는 부분도 마찬가지로 post를 삭제하면 plan이 cascading 되어 삭제된다.

위 예제는 한 트랜잭션에서 발생했기 때문에 실제로는 save() 함수가 동작하지 않는 것은 참고. 따로따로 두번의 트랜잭션으로 작업해봐야 예제가 정상적으로 돌거다.

### JPA 프로그래밍 6. Fetch

연관관계의 엔티티를 어떻게 가져올 것이냐... 지금가져온다면 (Eager) 나중에 가져온다면 (Lazy)

기본적으로 @OneToMany 어노테이션은 Lazy 방법으로 가져온다. 예를 들어 위에서 구현한 Post 객체와 Comment 객체로 참고해보면.
Post 객체를 가져오는 것은 단일 건인데 Comment 건은 복수개로 얼마나 많은지를 알수가 없다. 이러한 상황에서 Post 만가져와도 되는 비지니스 로직이라면 많은 수를 가지고 있는 Comment 객체를 가져오는 것은 비효율적이기 때문에 Lazy loading 방법으로 가져온다. 반대로 @ManyToOne 은 EAGER 방식이다.

```java

@OneToMany(fetch = FetchType.LAZY)

@ManyToOne(fetch = FetchType.EAGER)


Post post = session.get(Post.class, 1L);
System.out.println(post.getTitle());

Comment comment = session.get(Comment.class, 2L);
System.out.println(comment.getComment());
System.out.println(comment.getPost().getTitle());

```

session.get() 함수는 해당하는 값이 없는 경우 NULL 리턴, session.load() 는 없는 경우 예외를 반환, 프록시로도 가져올 수 있따.

이걸 잘 조절해야 성능 이슈를 해결할 수 있다. 가장 흔한 N+1 의 문제.


### JPA 프로그래밍 7. 쿼리

지금까지는 HIBERNATE API 에서 제공하는 Session 객체를 활용해서 구현을 했음.
JPA가 하이버네이트를 감싸고 있다. 내부적으로 사용하고 있따.

#### JPQL (HQL)

Java Persistencec Query Language / Hibernate Query Language

쿼리 모양이 우리가 기존에 알고 있는 SQL과 굉장히 유사하다. 다만 다른 점은 데이터베이스 테이블 기준이 아니고. 엔티티 객체 모델 기반으로 쿼리를 작성한다.

```java

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        entityManager.createQuery("SELECT p FROM Post AS p");
    }
}

```
JPQL / HQL 은 기존 SQL이 아니기 때문에 데이터베이스 벤더에 독립적이다. 사용되어지는 데이터베이스 쿼리로 변환되어 질의가 실행된다.

쿼리를 어노테이션으로 지정해두고 불러와서도 사용이 가능하다.


* toString에 @ManyToOne 타입으로 있는 필드를 찍어놓으면. 필요없이 단순히 습관적으로 한거겠지만. toString 호출시 Comment 관련 쿼리도 날라간다. toString 때문에.

JPA, 하이버네이트 쓸때는 항상 무슨 쿼리를 발생시키는지, 내가 성능이슈를 점검해야하니 의도한건지를 항상 파악해야한다. 학습비용이 드는거지만 어쩔수 없다.

JPQL 의 단점은 TypeSafe 하지 않다. 문자열로 전달되기 때문. -> 타입세이프한 방법이 있따 Criteria 사용해보자

```java

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;
import java.util.List;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Post> query = builder.createQuery(Post.class);
        Root<Post> root = query.from(Post.class);
        query.select(root);

        List<Post> posts = entityManager.createQuery(query).getResultList();
        posts.forEach(System.out::println);
    }
}

```

native Query 를 사용하는 방법도 있따.

```java

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;

@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<Post> posts = entityManager.createNativeQuery("Select * from Post", Post.class)
                .getResultList();

        posts.forEach(System.out::println);
    }
}

```

### 스프링 데이터 JPA 원리

EntityManager 를 직접 쓰던 상황에서 이제 Repository 라는 DAO 유사한 녀석을 만들고 여기 안에서 이 EntityManager 활용한 디비 작업을 모으기 시작함. 이걸 또 제네릭화 시키고 최종적으로 JpaRepository<T, ID> 형태로 공통화시켜 이걸 상속받기만 하면 되는 구조가 되어있음.

@EnableJpaRepositories 어노테이션은 Repository 관련 빈들을 자동 등록해주는 어노테이션인데 스프링 부트에는 자동설정을 해주기 때문에(Auto Configuration 방법으로 ). 이 어노테이션을 붙이지 않아도 되고. 또 각 Repository에 @Repository 어노테이션을 붙이지 않아도 됨.

기존에 EntityManager 을 활용해서 사요하는 방식보다 이 Spring Data JPA는 결국 Repository 부분을 보면 기본적인 CRUD 쿼리들은 제공이 되기 때문에 이를 사용하고 되면 이 말은 즉은 내가 만든 코드가 아니기때문에 테스트 영역 줄어둔다. 코드가 적으니 생산성이 살아나고, 유지보수도 좋아진다.

@EnableJpaRepositories 어노테이션 안을 살펴보면 JpaRepositoriesRegistrar 클래스를 임포트 하고 있는데 이녀석이 바로 JPA Repository 들을 주입해주는 역할을 한다. 요 녀석은 JpaRepository 를 상속받는 Repository 들을 찾아서 빈으로 등록해준다. 이를 예제로 코딩해보면 아래와 같다.

```java

public class Keesun {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.beans.factory.support.GenericBeanDefinition;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;

public class KeesunRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(Keesun.class);
        beanDefinition.getPropertyValues().add("name", "whiteship");

        registry.registerBeanDefinition("keesun", beanDefinition);
    }
}

@SpringBootApplication
@Import(KeesunRegistrar.class)
public class Demojpaspringdata2Application {

    public static void main(String[] args) {
        SpringApplication.run(Demojpaspringdata2Application.class, args);
    }

}

```

보면 Keesun 클래스가 빈 등록을 위한 어노테이션이 없음에도 KeesunRegistrar 객체에서 Keesun 객체를 프로그래밍적으로 빈 등록을 해주고 있따.
내가 만든 Repository 클래스들도 위와 같은 방식으로 JpaRepositoriesRegistrar 요 객체가 등록을 해준다.

물음표로 나오는 값을 실제 값이 나오게끔 로깅하는 방법.

```
logging.level.org.hibernate.type.descriptor.sql=trace
```

### 스프링 데이터 JPA 활용

Spring Data 라는 것은 여러 개의 프로젝트를 범주잡아 일컫는 말임. 여기에 JPA, REDIS, REST, MONGODB 등 많은 것들이 포함되어 있음.
그중 공통적인 작업들을 Spring Data Common 이라는 프로젝트에 넣어뒀는데 여기에 리포지토리를 생성하고, 메소드 쿼리를 구현하는 작업들이 있음.

### Spring Data Common 1. 리포지토리

Spring Data Common 프로젝트에 있는 3개의 상위 리포지토리 - Repository, CrudRepository, PagingAndSortingRepository
Spring Data Jpa 프로젝트에 있는 리포지토리 - JpaRepository.

@NoRepositoryBean 어노테이션이 붙는 이유 - 이게 붙어있는 리포지토리는 빈으로 등록하는 걸 방지 즉 서브 클래스를 만들어서 상속받아서 사용하라는 이야기.

@DataJpaTest DAO 레벨을 테스트할 떄 이 어노테이션을 스프링부트에서 제공해준다. -> Repository 만 등록이 된다.
리포지토리만 빈등록 스코프가 잡히기 때문에 빈이 많은 프로젝트의 경우 보다 빈등록하는데에 가볍다.

h2 디비는 메모리 디비를 사용해서 테스트를 하면 실제 어플리케이션에서 사용하는 포스트그레스큐엘 디비에는 영향이 가지 않는다.
또 H2 디비는 메모리 디비 이기 떄문에 실제 디비를 사용하는것보다 더 빠르다.

```xml

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>

```

import static org.assertj.core.api.Assertions.assertThat 이 라이브러리르 사용하면 아래처럼 코드를 짤 수 있다.
assertThat(post.getId()).isNull(); 

기본적으로 @Test 테스트 코드는 테스트 이후에 롤백이 된다. 이 트랜잭션 처리하는 롤백기능은 스프링 프레임워큭에서 제공하는 기능.
롤백을 하고싶지 않다고 하면 어노테이션 추가.

```java

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    @Rollback(false)
    public void crudReposiroy() {
        // Given
        Post post = new Post();
        post.setTitle("hello spring boot common");
        assertThat(post.getId()).isNull();

        // When
        Post newPost = postRepository.save(post);

        // Then
        assertThat(newPost.getId()).isNotNull();

        // When
        List<Post> posts = postRepository.findAll();

        // Then
        assertThat(posts.size()).isEqualTo(1);
        assertThat(posts).contains(newPost);

        // When
        Page<Post> page = postRepository.findAll(PageRequest.of(0, 10));

        // Then
        assertThat(page.getTotalElements()).isEqualTo(1);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getSize()).isEqualTo(10);
        assertThat(page.getNumberOfElements()).isEqualTo(1);

        // When
        postRepository.findByTitleContains("spring", PageRequest.of(0, 10));

        // Then
        assertThat(page.getTotalElements()).isEqualTo(1);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getSize()).isEqualTo(10);
        assertThat(page.getNumberOfElements()).isEqualTo(1);

        // When
        long spring = postRepository.countByTitleContains("spring");

        // Then
        assertThat(spring).isEqualTo(1);
    }
}

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long> {

    Page<Post> findByTitleContains(String title, Pageable pageable);

    long countByTitleContains(String title);
}

```

위 코드는 기존에 만들어져 있는 CrudRepository 와 PagingAndSortingRepository 기능들을 테스트 코드 짜본 것. 의미는 없는데 내가 보기엔 Repository 테스트코드를 이런식으로 작성해야 한다를 보여주는 것 같음.

쿼리 메소드

H2 DB를 사용하고 있는지는 스프링 부트가 뜰때 로그를 보면 알수 있다. 단 @DataJpaTest 어노테이션을 붙여야함. 그리고 의존성을 추가했을 떄.


```
2020-12-25 22:26:27.671  INFO 3476 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialectㅌㅈ
```