---
layout: post
title:  "JPA Entity 관련 annotation 에 대한 고찰"
date:   2020-12-28 11:38:54 +0900
categories: spring
---

### 기본 어노테이션

이 글은 백기선 개발자님의 Spring Data JPA 강의를 듣고 JPA 관련 내용들을 정리한 글이다. 기본적으로 해당 강좌에서 언급되는 개념으로 진행되지만, 저자의 JPA 경험을 토대로 이해한 바가 함께 녹아있으므로 그 부분은 참고하고 글을 읽어주시기 바란다.

JPA에서 객체와 릴레이션간에 매핑을 위해 메타데이터를 생성하는 방법은 어노테이션 방법과 xml방법 두 가지가 있다. 그런데 요즘은 거의 어노테이션 방법만 사용하고 있다. 파일이 구분되지 않고 가독성이 높아지기 떄문인데 이는 스프링 설정 방법이 어노테이션 방식으로 바뀌어 가는 것과 동일하다고 본다.

@Entity 어노테이션에 설정한 이름은 객체 세상에서만 사용되어지는 거고 테이블의 이름을 바꾸고 싶으면 @Table 어노테이션을 사용해야 한다. @Table 어노테이션은 릴레이션 관련 정보를 설정하는 어노테이션이다. 테이블의 이름을 변경하고 싶다면 이 어노테이션에 옵션으로 주면 된다. 그런데 이 어노테이션은 @Entity 어노테이션과 옵션을 바라보고 있기 때문에 @Entity 어노테이션의 옵션으로 이름을 변경하게 되면, @Table 어노테이션의 이름도 마찬가지로 변경된다. 

@Id 는 주키를 설정하는 어노테이션. 문서 상에는 모든 타입을 제공한다. 여기서 논쟁거리 하나가 int, long과 같은 원시타입을 써야하는지 혹은 객체타입을 써야하는지에 대한 이슈이다. 백기선 강사님은 객체타입을 사용하는 걸 추천하는데 숫자를 원시타입으로 사용할 경우 초기 값에 0이 들어가게 되고, 이 0 값이 특정 비지니스에서는 필요한 값일 수도 있기 때문에 초기 값을 null을 넣는 걸 권장하기 때문이다.  

```java

/* primitive type */
@Id
private long id;

/* reference type */
@Id
private Long id;

```

@GeneratedValue 

어노테이션은 자동 생성된 값을 사용하겠다는 어노테이션이다. 자동 생성 규칙은 각 데이터베이스의 규칙에 따라서 생성이 되며, SEQ를 쓸건지, Identity 객체를 쓸건지..  명시적으로 옵션을 통해 설정할 수 있디. 예를 들면 @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = '') 이런 식이다.

@Column 

도메인 객체 각 필드에 사용하는 어노테이션이다. 옵션으로 nullable, unique, name 등을 줄수 있다.

@Temporal 

날짜와 시간에 관련된 필드에 매핑이 가능하다. 3가지 속성이 있으며 상황에 맞춰서 쓰면 된다. LocalDate는 자바 1.8 이후에 생겨난 객체이기 떄문에 JPA 2.1 이전 버전에는 이 타입에 대한 매핑이 지원이 안된다. 따라서 Calendar와 Date 객체를 사용해야 한다. 그렇기 떄문에 LocalDate 객체를 사용해야 한다면 JPA 2.1 이상 버전을 추천한다.

@Transient 

이 어노테이션을 사용하면 사용된 필드는 릴레이션의 컬럼으로 매핑을 하지 안겠다는 의미이다. 이 용어는 JPA에서 관리하는 엔티티 상태 4가지 중 하나와 동일하다.

### 참조타입 필드를 위한 어노테이션

만약 필드에 기본 타입이 아니고 객체타입을 사용하고 싶은 경우에는 @Embeddable, @Embedded 어노테이션을 활용해야 한다.
예를 들어 주소 관련 정보를 가지고 있는 엔티티를 만든다고 하자. 이 주소 정보는 도로명을 가지고 있는 street 필드, 도시명을 가지고 있는 city 필드, 주명을 가지고 있는 state, 우편번호를 가지고 있는 zipCode 이렇게 4개가 필요하다.
이 정보들을 한 엔티티에 바로 추가할 수 있지만. 관련된 필드들을 묶고싶어서 Address라는 객체를 만들고 이 객체 타입으로 필드에 넣었다.

이렇게하면 실제 엔티티에는 이 4개의 필드 정보가 직접적으로 생성된다. 객체관점에서는 구분되어있지만 릴레이션 관점에서는 구분되어있지 않는다는 것이다.

엔티티로서 사용해야할 만큼 큰 객체라면 @Entity 어노테이션을 써서 기존처럼 새로운 릴레이션을 만들면 된다. 그러나 지금처럼 Address의 경우 그정도의 사이즈는 아니고 단순하게 값을 묶어서 가지고 있는 정도이기 때문에 이 녀석을 @Embeddable, @Embedded 어노테이션을 활용하면 객체에서는 묶여있지만 릴레이션에서는 모두 포함된 구조로 구현이 가능하다.

```java

@Getter
@Setter
@Embeddable
public class Address {
    private String street;

    private String city;

    private String state;

    private String zipCode;
}

@Getter
@Setter
@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @Embedded
    private Address address;
}

```

같은 Embeddable 객체를 한 엔티티 내에 여러번 사용할 경우에는 @AttributeOverrides, @AttributeOverride 어노테이션을 활용해 이름을 변경할 수 있다. 예를 들면 집주소, 직장주소 등이 있다.

### 관계 어노테이션

엔티티끼리의 관계가 있을 때 릴레이션에게 이 관계를 설명하기 위한 메타데이터를 설정해야 한다. 관계가 있다는 것은 객체관점에서는 의존성을 가지고 있다는 것이고 릴레이션 관점에서는 한쪽 방향에서는 FK를 가지고 있다는 의미이다.

1:N 관계에 대한 매핑을 하기 위해서는 @ManyToOne 어노테이션을 사용해야한다. Account 객체와 Study 객체로 예를 들어 생각해보자. 아래 예시를 보면 Study 객체에 Account 필드를 생성하고 여기에 @ManyToOne 어노테이션을 붙였다. 이렇게 하면 데이터베이스에서는 Account 릴레이션의 PK를 참조하는 FK 값이 Study 릴레이션에 생성된다. Study가 Account를 의존하고 있기 때문에 Study는 이 FK를 활용해서 Account에 접근이 가능하다.

```java

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Getter
@Setter
@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account ownwer;
}

```

이번엔 Account 객체 기준으로 Study에 접근한다고 생각해 보자. Account 한 개가 여러 개의 Study를 가질 수 있으므로 아래와 같이 컬렉션 타입으로 Study 필드를 생성하고 @OneToMany 어노테이션을 붙인다. @ManyToOne 방식은 FK를 주는 것으로 구현이 되었다면 @OneToMany 어노테이션이 릴레이션에 매핑될 때에는 기본적으로 조인테이블이 생성된다.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Getter
@Setter
@ToString
@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany
    private Set<Study> studies = new HashSet<>();
}

```

양방향 참조를 하려면 어떻게 해야할까? 즉 Studt 객체도 Account 객체를 의존하고 있고 Account 객체도 Study 객체를 의존하고 있는 구조다. 사실 이 구조는 디자인 패턴 관점에서는 권장하지 않는 구조인데. 이렇게 되면 Account 객체와 Study 객체간 결합성이 너무 높아지기 때문이다. 여하튼 그럼에도 이런 구조를 가지고 있을 때 양방향성을 릴레이션에 매핑하려면 우선 위 처럼 @OneToMany, @ManyToOne 어노테이션을 붙인다. 그리고 Account 객체에서 @OneToMany 속성 중 mappedBy에 Study 객체가 Account 객체를 참조하기 위해 사용한 필드의 이름을 넣어준다.

```java

@Getter
@Setter
@ToString
@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();
}

@Getter
@Setter
@ToString
@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account owner;
}

```

이렇게 양방향성을 가진 두 객체에 실제로 데이터를 넣을때에는 양 객체에 모두 쿼리를 날려야 한다.
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
        /* Account 관련 데이터 INSERT */
        session.save(account);
        /* Study 관련 데이터 INSERT */
        session.save(study);
    }
}

```

양방향성을 가진 객체끼리는 항상 쿼리를 날릴 때 위와 같이 두 객체를 위한 쿼리를 따로 날려야 하기 때문에 보통 아래 addStudy(), removeStudy() 함수와 같이 이들을 함께 호출한다.

```java

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Getter
@Setter
@ToString
@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();

    public void addStudy(Study study) {
        this.getStudies().add(study);
        study.setOwner(this);
    }

    public void removeStudy(Study study) {
        this.getStudies().remove(study);
        study.setOwner(null);
    }
}

```