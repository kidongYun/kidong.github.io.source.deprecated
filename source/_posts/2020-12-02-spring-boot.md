---
layout: post
title:  "Spring Boot"
date:   2020-12-02 11:38:54 +0900
categories: spring
---

### 스프링부트 소개

제품 수준의 스프링 프레임워크 기반의 제품을 만들때 빠르게 만들수 있다.

스프링에서 가장 널리 쓰이는 설정을 기본적으로 설정한 것. (convention of configuration)
-> 원한다면 보다 더 쉽게 커스터마이징도 가능.

third-party 라이브러리에 대한 처리도 어느정도 제공 -> ex) 톰캣이 내부적으로 뜬다.

스프링 부트는 자바 8 이상부터 사용이 가능.

## 스프링부트 프로젝트 생성

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SpringBootLecture</artifactId>
    <version>1.0-SNAPSHOT</version>

<!--    스프링 부트 프레임워크가 부모 의존성 임을 명시-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

    <repositories>
        <repository>
            <id>nexus-central</id>
            <name>Interpark Nexus Central Maven Repository</name>
            <url>http://nexus.interpark.com:8888/nexus/content/repositories/central/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>

```