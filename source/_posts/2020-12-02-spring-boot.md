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

dependencies 항목들은 repositories 에서 의존성을 가져오고 plugin 들은 pluginRepositories에서 가져온다.



### Spring-boot maven 활용 직접 세팅하는 방법

pom.xml 설정

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SpringBootAutoConfigure</artifactId>
    <version>1.0-SNAPSHOT</version>

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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
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
    <pluginRepositories>
        <pluginRepository>
            <id>nexus-central</id>
            <name>Interpark Nexus Central Maven Repository</name>
            <url>http://nexus.interpark.com:8888/nexus/content/repositories/central/</url>
        </pluginRepository>
    </pluginRepositories>
</project>

```

2. 원하는 패키지 생성하고 거기에 Application.java 파일 생성


Application.java

```java

package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

### 스프링 부트 프로젝트 구조.

```
    src
        main
            java
            resource
        test
            java
            resource
```

main application의 추천하고 있는 위치는 사용하고 있는 최상위 default package 에 넣는걸 추천.
-> @ComponentScan 이 붙어있기 때문에 그렇다. 이 파일이 들어있는 패키지 기준으로 컴포넌트를 스캔하기 때문.

하나 주의해야할 점은. 이 메인 어플리케이션이 하위에 있는 자바 파일들만 빈등록이 되는것. 그 외 패키지에 있는것은 등록이 안됩니다.!!!

@SpringBootApplication 이 붙은걸 메인 어플리케이션이라고 부름.


### 스프링 부트 의존성 원리

pom.xml 안에 <parent> 태그에 우리는 아래처럼 'spring-boot-starter-parent' 을 받고있다. 여기 안에 들어가 보면 또 부모가 있는걸 볼수 있는데 그 녀석 이름이 'spring-boot-dependencies' 이다. 여기에 또 들어가보면 기본적으로 Spring 을 사용하는데 필요한 의존성들을 모두 주입해둔것을 볼 수 있다. dependencyManagement 라는 태그로 되어있음.

```xml

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

```

이렇게 의존성을 스프링 부트 내부적으로 관리해주고 우리는 spring-boot-starter-web 이런식으로 한두개만 의존성 관리할게 될때의 장점은. 버전을 올리거나 내려야 할 상황이 올때
모든 라이브러리들이 호환이 잘되는지 않되는지를 명확히 알기 어렵기도 하기 때문에 이런 버전 관리가 어렵다. 그래서 이렇게 관리해주고 있는 이러한 스프링 부트만을 의존성을 받게 되면.
이미 관리되어진 의존성만을 가져오는 거기때문에 버전 문제에 대한것을 쉽게 처리할 수 있다.

<parent/> 태그를 사용하지 않고 의존성을 관리하는 방법도 있다.

<parent/> 가 만약 다른 걸로 고정이되어서 수정할 수 없느경우 <dependencyManagement/> 이 태그를 사용해서 보다 더 커스터마이징된 의존성을 넣을 수 있다. 여기에 spring-boot-dependencies를 넣으면 됨.

단점 -> parent 쪽에서 들어오는거가 dependency 만 들어오는게 아니고 다른 속성들 (인코딩, 자바 버전, 플러그인 설정, yml 파일 사용) 도 설정을 해주는 거기 때문에 이런것들을 모두 관리해야 한다.
parent에서 의존성 말고 관리해주느 것들을 <parent/> 이 태그로 못가져 오기 때문에. 이 부분들을 모두 나의 프로젝트에서 관리해주어야 한다.


### 스프링 부트 의존성 관리 응용

mvnrepository -> maven 의존성의 허브사이트.

1. 의존성을 추가하는 방법

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
```

modelMapper -> domain을 가지고 flatDesign을 만들어준다

```xml
        <dependency>
            <groupId>org.modelmapper</groupId>
            <artifactId>modelmapper</artifactId>
            <version>2.3.0</version>
        </dependency>
```

version은 항상 명시해줘야한다. Spring Boot가 지원해주는 애들은 version이 자동 관리가 되긴 함 이거는 intellij 소스창 옆에 보면 동그랗게 뜸


```xml
    <properties>
        <spring.version>5.0.6.RELEASE</spring.version>
    </properties>
```

스프링 버전을 바꾸고싶다면 properties 태그에 위와같이 추가한다. 이는 Spring-boot parent 에 dependencies parent 에 가보면 동일한 형태로 정의가 되고 있음을 볼수 있다.
자바 버전을 바꾸고싶다면 properties 태그에 똑같이 아래와 같이 수정하면된다.

```xml
    <properties>
        <java.version>1.8</java.version>
    </properties>
```

### 자동 설정 이해 (EnableAutoConfiguration)

@SpringBootApplication 는 중요한 3개의 어노테이션을 합친것

@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration


스프링 부트는 2단계로 의존성을 가져온다?
ComponentScan + EnableAutoConfiguration

@EnableAutoConfiguration 이 없어도 어플리케이션을 실행할 수는 있다. 그러나 이것은 웹서버는 없다. 웹어플리케이션에 대한 초기 자동 설정을 버리고 그냥 스프링 프레임워크만 띄웠다는 의미인듯.

```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        application.run(args);
    }
}

```

@ComponentScan 으로 빈을 먼저 등록한 다음에
@EnableAutoConfiguration 이걸로 추가적인 빈 등록을 한다 2단계로 나누어진다.

@ComponentScan -> 스프링의 기본적인 내용
    @Component, @Configuration @Repository, @Service @Controller @RestController 등등의 어노테이션이 되어있는 인스턴스를 리플렉션으로 생성해서 스프링 컨테이너에 빈으로서
    넣어두는거 같음. 기본적인 내용.
    자기 어노테이션 기준으로 하위 패키지에 저 어노테이션들이 박혀있는 클래스들을 인스턴스화 후 빈 등록.

@EnableAutoConfiguration -> Spring meta 파일을 보고 가져온다.
org.springframework.boot:spring-boot-autoconfigure 프로젝트 안에
META-INF 폴더가 있고 이안에 spring.factories 라는 파일이 있다.
거기 안에 org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ 라는 경로로 키가 설정 되어있오 그밑으로 Configuration value 값들이 들어있는데 이걸 다 가져오는거 같아.
여하튼 이 파일을 보고 여기에 설정되어있는 값들을 모두 빈등록.

WebMvcAutoConfiguration 이 클래스도 EnableAutoConfiguration 으로 빈 등록되는 객체인데
그 안에보면 @Conditional.... 조건에 따라서 등록되고 안되는것도 볼수있다.

모두 빈 등록을 하지만 조건에 따라 안되는것도 있음을 참고해야함.


### 자동 설정 만들기 1부: Starter와 AutoConfigure

Xxx-Spring-Boot-Autoconfigure => 자동설정 등을 설정할때 이런식으로 이름을 만들고 프로젝트를 만든다.
Xxx-Spring-Boot-Starter => 필요한 의존성등을 정의할때 보통 이런식으로 이름을 만든다.
그냥 하나로 하고싶을 때에는 Starter 이름을 사용하자.

```java

/* Holoman.java */

package me.whiteship;

public class Holoman {
    String name;
    int howLong;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getHowLong() {
        return howLong;
    }

    public void setHowLong(int howLong) {
        this.howLong = howLong;
    }

    @Override
    public String toString() {
        return "Holoman{" +
                "name='" + name + '\'' +
                ", howLong=" + howLong +
                '}';
    }
}

```

```java

/* HolimanConfiguration.java */
package me.whiteship;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HolomanConfiguration {

    @Bean
    public Holoman holoman() {
        Holoman holoman = new Holoman();
        holoman.setHowLong(5);
        holoman.setName("Keesun");
        return holoman;
    }
}

```

src/main/resource/META-INF에 spring.factories 파일 생성

이 파일은 스프링부트가 아니고 스프링에서 원래 제공해주는 형식의 파일이다 여기서 라이프사이클도 조절이 가능하다.

```factories

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  me.whiteship.HolomanConfiguration

```

그 다음에 다른 프로젝트에서 이 프로젝트를 사용할 수 있도록 mvn install 해서 빌드해서 jar 파일로 만들고 이걸 로컬 메이븐 저장소에다가 등록 다른 lib에 등록해준다.

사용하려고하는 프로젝트를 열고 의존성 추가 의존성 groupId 와 artifactId, version 등은 Holoman 프로젝트 pom.xml 에서 확인할 수 있다.

```xml

    <dependencies>
        <dependency>
            <groupId>me.whiteship</groupId>
            <artifactId>kidong-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    
```

ApplicationRunner 는 스프링 프레임워크가 시작되었을 때 자동으로 실행되는 빈을 만들고 싶을때 사용하면 된다. 인터페이스 객체임으로 implements 받아서 사용가능

```java

package com.example;

import me.whiteship.Holoman;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class HolomanRunner implements ApplicationRunner {

    /* 이 프로젝트에서는 어디에서도 Holoman 객체를 빈으로 등록하지 않았다 */
    @Autowired
    Holoman holoman;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(holoman);
    }
}

```

이 방법의 문제는 내가 직업 위처럼 자동설정한 빈을 아래처럼 직접 다시 생성하면 생성한 빈이 무시가 된다. 

```java

package com.example;

import me.whiteship.Holoman;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        application.run(args);
    }

    @Bean
    public Holoman holoman() {
        Holoman holoman = new Holoman();
        holoman.setName("whiteship");
        holoman.setHowLong(60);
        return holoman;
    }
}

```

왜냐하면 빈 등록을 하는 단계가 2개라고 이전에 언급했는데
@ComponentScan 방법이 먼저이고 @EnableAutoConfiguration 인데 직접 생성한 빈 설정은 @ComponentScan 방법에서 만들어졌다가 2단계에서 묻히는것.
@EnableAutoConfiguration 요거에 특정 케이스에는 빈 등록을 안하게끔 하는 설정들이 있엇다 그걸 활용하면 이 문제를 개선 가능하다.
내 설정보다 자동설정이 더 우선된다.

최근 버전으로 살펴보니 아예 오류가 떨어진다.

```
The bean 'holoman', defined in class path resource [me/whiteship/HolomanConfiguration.class], could not be registered. A bean with that name has already been defined in com.example.Application and overriding is disabled.
```


### 자동 설정 만들기 2부: @ConfigurationProperties


