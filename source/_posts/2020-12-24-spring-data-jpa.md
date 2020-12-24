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


