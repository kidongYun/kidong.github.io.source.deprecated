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
2.  