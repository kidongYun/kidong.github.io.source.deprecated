---
layout: post
title:  "JPA 기본 내용에 대한 고찰"
date:   2020-12-24 11:38:54 +0900
categories: spring
---

### JPA 프로그래밍 5. 엔티티 상태와 Cascade

Cascade 속성은 @ManyToOne, @OneToMany 어노테이션 옵션으로 가지고 있는데 이 어노테이션의 의미는 해당 엔티티 상태의 변화를 어노테이션이 설정된 필드에도 전파하겠다라는 의미이다.
용어 자체의 의미는 사전적 의미와 데이터베이스에서 사용되던 의미랑 유사한것 같다.

```java

@Entity
public class Study {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(cascade = Cascade.PERSIST)
    private Account owner;
}

```

예를 들어서 위 코드에서 Study Entity가 A 상태에서 B 상태로 전이될때 Account 객체에도 이 전이를 전파하고 싶다면, 위와 같이 cascade 옵션으로 제공하면 된다. 그러면 전파되는 Entity의 상태라는 것이 어떤 것들이 있는지 알아보자. Entity에는 Transient, Persistent, Detached, Removed 로 총 4가지의 상태가 있다. 이에 대해여 간략하게 알아보도록 하자.

#### 1. Transient
```java

public void run() {
    Account account = new Account();
    account.setUserName("keesun");
    account.setPassword("jpa");
}

```

위 코드에서는 Account 타입의 객체를 생성하고 필드에 값을 주입하고 있다. 순수하게 Account 객체를 만들고 이를 데이터베이스에 반영하기 위해 JPA에 전달하는 업무가 없다. 이런 작업처럼 릴레이션쪽에 반영이 되는 것이 없고 오직 자바 객체 수준에서만 변화가 있는 상태를 Transient 라고 한다. JPA는 이러한 작업들이 데이터베이스에 영향을 주지 않기 때문에 자신의 관리포인트 안에 두지 않는다. Entity 클래스에서 필드 내에 Transient 어노테이션을 넣는 것과 개념면에서 유사하다. 

2. Persistent

```java

session.save(account);

```

위에서 만든 account 인스턴스를 하이버네이트 세션을 통해 영속화를 한 이후에는 Persistent 상태로 바뀐다. save를 했다고 해서 바로 데이터베이스에 들어가는 것은 아니다 우선 JPA의 Persistent Contenxt 라는 곳에서 캐싱을 하고 있다가. JPA 스스로 데이터베이스에 들어가야 한다고 판단하는 시점에 들어간다. 이 Persistent Context 영역은 JPA는 데이터베이스에 들어가거나 나와야할 데이터들을 관리하며 쿼리 요청시 해당 데이터가 질의가 필요한지 아닌지도 검증하는데 사용한다.

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

위의 예제를 보면서 Persistent Context 어떤식으로 동작하는지 살펴보자. Study, Account 타입의 새 객체를 만들고 이를 save() 함수를 통해 데이터베이스에 저장했다. 그 이후에 저장된 데이터를 load() 함수를 통해서 다시 꺼내고 이 데이터 내용을 표준 콘솔로 출력하고 있다. 이런 경우 save() 함수를 호출 했을 때 들어간 데이터들을 JPA는 Persistent Context 영역에 캐싱을 해둔다. 그렇기 때문에 만약 동일한 데이터를 load() 함수로 호출하게 되면 데이터베이스에 접근해서 해당 데이터들을 가져오는 것이 아니고 캐싱된 Persistent Context에서 가져온다.
이를 활용하게 되면 실제 필요한 질의의 수가 줄어들기 때문에 성능 향상을 기대해볼 수 있다.

추가적으로 save() 함수 호출했을 때 데이터를 바로 INSERT 하는 것은 아니다. 보통 한 트랜잭션이 끝난 경우에 INSERT 쿼리가 발생하고 실제 데이터베이스에 반영이 된다.

이번에는 Persistent 상태가 가지는 특성 중 하나인 Dirty Checking을 알아보자

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
2020-12-25 22:26:27.671  INFO 3476 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
```

### Spring Data Common 2. 인터페이스 정의하기

지금까지는 스프링데이타 Common 이나 스프링 데이터 JPA 에서 제공하는 리포지토리의 기능이 들어오는게 싫다. 내가 다 정의하고 싶은 경우.

```java

import org.springframework.data.repository.RepositoryDefinition;
import java.util.List;

@RepositoryDefinition(domainClass = Comment.class, idClass = Long.class)
public interface CommentRepository {

    Comment save(Comment comment);

    List<Comment> findAll();
}

```

이렇게 @RepositoryDefinition 어노테이션을 활용해서 직접 정의가 가능하다. 이런식으로 정의했을 때 공통적으로 쓰이는 기능들을 묶고싶다면. Repository 객체의 최상위 인터페이스인 Repository를 상속받고 이를 통해 구현하도록 하자.

```java

import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.Repository;

import java.io.Serializable;
import java.util.List;

@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends Repository<T, ID> {

    <E extends T> E save(E Comment);

    List<T> findAll();
}

public interface CommentRepository extends MyRepository<Comment, Long> {

}

```
JpaRepository 같은걸 커스텀하게 만들었다고 생각하면 되겠다.

### Spring Data Common 3. Null 처리

단일 값을 받을때 Optional 로 받는것을 권장. List로 받는것은 Null이 안나온다. 아마 갯수는 없는 List의 형태의 객체가 나올듯(비어있는 콜렉션). 그렇기때문에 List 로 받는 것은 Optional 로 받을 필요가 없음. 이건 Spring Data JPA 특징. 이걸 통해서 컬렉션 구조를 반환하는 함수는 결코 NULL이 되지 않음.

@NonNull, @Nullable

```java

import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.Repository;
import org.springframework.lang.NonNull;
import org.springframework.lang.Nullable;

import java.io.Serializable;
import java.util.List;
import java.util.Optional;

@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <E extends T> E save(@NonNull E Comment);

    List<T> findAll();

    long count();

    @Nullable
    <E extends T> Optional<E> findById(ID id);
}

import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataJpaTest
class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
        commentRepository.save(null);
    }
}

```

파라미터에 Null이 들어가면 안될 때 해당 파라미터에 @NonNull 어노테이션을 붙이면 IDE에서 점검이 가능하고 런타임시에 Null이 들어갔다고 exception이 발생한다.

### Spring Data Common 4. 쿼리 만들기

쿼리 만드는 방법은 2가지가 있다.

@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.CREATE)
1. 쿼리 메서드 (메소드 이름을 분석해서 쿼리 만들기)

@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.USE_DECLARED_QUERY)
2. @Query 어노테이션 활용 (미리 정의해 둔 쿼리 찾아 사용하기.) 기본값은 JPQL 이며 SQL 을 사용하고 싶다면 옵션중에 nativeQuery를 true로 적용.

이거가 디폴트 맞네
3. 미리 정의한 쿼리를 찾아보고(메서드 쿼리를 찾아보고) 없으면 만들기. -> 이거를 사용하고 싶으면 @EnableJpaRepositories 이 어노테이션에 
@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND) 이렇게 옵션을 주면 됨.

쿼리 찾는 방법.

@Query

@Procedure

@NamedQuery

```

메서드 쿼리 사용시 규칙

접두어 : Find, Get, Query, Count

도입부 : Distinct, First(N), Top(N) /* 생략 가능* /

프로퍼티 표현식 : 

```

보통 리포지토리는 한 엔티티를 위한 리포지토리로 구현이 되어야함. 그렇기 때문에 내가 생각했던 그 JpaBeanRepository를 만드는 것은 모두를 위한 리포지토리를 만드는 것이기 때문에 내가 볼때는 Spring Data jpa의 정책에 조금 어긋난 방법이지 않았나. 그래서 서비스단에서 Repository를 기본 CRUD에 대해서 하나로 묶고 싶다면 JpaService 하나에 Repository를 여러개 넣는 방법이 최선일것 같고.

Pageable 객체에 Paging 관련 기능과 sorting 관련 기능이 같이 있다 이걸 활용하면되고, sorting만 해야할 경우에는 Sort 객체 활용. 보통 쿼리를 만들때에는 Pageable를 권장.



(1) 메서드 쿼리로 구현이 가능한지 확인

확인하는방법 -> 테스트를 만들어서 확인하면됨

(2) 메서드 쿼리를 잘 만든건지 확인 -> 그냥 돌려보면 된다ㅏ. 비어있는 테스트 함수 하나 만들고 돌리면 됨.

(3) 쿼리DSL 사용

### Spring Data Common 5. 쿼리 만들기 실습

ignoreCase 를 쓰면 upper() 쿼리가 추가되어 대소문자 구분이 사라진다.

Stream<> 타입으로 받으면 try-with-resouce 문법을 사용할 것 Stream을 다쓴다음에 close() 해야함.

```java

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.util.List;

public interface CommentRepository extends MyRepository<Comment, Long> {

    List<Comment> findByCommentContains(String keyword);

    List<Comment> findByCommentContainsIgnoreCase(String keyword);

    List<Comment> findByCommentContainsIgnoreCaseAndLikeCountGreaterThan(String keyword, Integer likeCount);

    List<Comment> findByCommentContainsIgnoreCaseOrderByLikeCountDesc(String keyword);

    List<Comment> findByCommentContainsIgnoreCaseOrderByLikeCountAsc(String keyword);

    Page<Comment> findByCommentContainsIgnoreCase(String keyword, Pageable pageable);
}

```

### Spring Data Common 7. 커스텀 리포지토리 만들기

255자 이상의 컬럼은 @Lob 어노테이션을 넣어주면 됨.

#### 스프링 데이터 리포지토리 인터페이스 기능에 추가

커스텀 리포지토리는 JPA, SPRING 에 침투받지 않음. 순수한 POJO 객체임 이거를 구현하는 Impl 클래스를 만들고 여기에서 EntityManager 를 통해 직접 데이터 액세스하는 부분을 구현한다.
그리고 커스텀 리포지토리를 JpaRepository를 상속하고 있는 비지니스별 Repository에 같이 상속시켜준다.



```java

import java.util.List;

public interface PostCustomRepository<T> {
    List<Post> findMyPost();

    void delete(T entity);
}

```

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository<Post> {
    @Autowired
    EntityManager entityManager;

    @Override
    public List<Post> findMyPost() {
        System.out.println("custom findMyPost");
        return entityManager.createQuery("SELECT p FROM Post AS p", Post.class).getResultList();
    }

    @Override
    public void delete(Post entity) {
        System.out.println("custom delete");
        entityManager.detach(entity);
    }
}

```

```java

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository<Post> {

}


```


Impl 이라는 용어가 싫으면 @EnableJpaRepositories 요 어노테이션에 repositoryImplementationPostfix 옵션에 원하는 문구로 바꾸면 됨.

### 스프링 데이터 Common 8. 기본 리포지토리 커스터마이징

모든 리포지토리에 공통적으로 추가하고 싶은 기능이 있거나 덮어쓰고 싶은 기본 기능이 있다면. 이렇게하자

우선 내가 했던대로 JpaRepository<T, ID> 요걸 상속하는 Repository를 하나 만든다. 다른점은 구현체를 하나더 만들고 이 구현체는 SImpleJpaRepository를 상속

entityManager.contains -> persistent context에 해당 객체가 있는지 없는지를 확인해주는 함수

JpaRepository 를 상속한 리포지토리는 @NoRepositoryBean 등록 바로 빈등록 하면 안된다고 얘기하는 것임

```java

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.repository.NoRepositoryBean;

import java.io.Serializable;

@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
    boolean contains(T entity);
}

```

```java

import org.springframework.data.jpa.repository.support.JpaEntityInformation;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;

import javax.persistence.EntityManager;
import java.io.Serializable;

public class SimpleMyRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {

    private EntityManager entityManager;

    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public boolean contains(T entity) {
        return entityManager.contains(entity);
    }
}

```

```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@SpringBootApplication
@EnableJpaRepositories(repositoryBaseClass = SimpleMyRepository.class)
public class Demojpa3Application {

    public static void main(String[] args) {
        SpringApplication.run(Demojpa3Application.class, args);
    }

}

```

```java

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {
    @Autowired
    PostRepository postRepository;

    @Test
    public void crud() {
        Post post = new Post();
        post.setTitle("hibernate");

        assertThat(postRepository.contains(post)).isFalse();

        postRepository.save(post);

        assertThat(postRepository.contains(post)).isTrue();

        postRepository.delete(post);
        postRepository.flush();
    }
}

```

이방법은 내가 생각한게 아니고. JpaRepository 가 기본적으로 제공하지ㅇ 않는 어떤 기능을 모든 Repository 들이 공통적으로 사용하게끔 하고 싶을때 사용하는 방법.

### Spring Data Common 9. 도메인 이벤트 퍼블리싱 기능

도메인 엔티티의 변화를 이벤트로써 발생시키고, 이벤트리스너가 이러한 변화를 감지하고, 이벤트 기반의 프로그래밍이 가능하게끔 하는 것.

ApplicationContext 는 우리가 아는 IoC의 기능을 하는 BeanFactory 를 상속을 받았고 또 ApplicationEventPublisher 도 상속을 받았다.

모든 스프링에는 이벤트 기반 코딩이 가능하다. 이를 지원한다.

우선 먼저 스프링에서 제공하는 이벤트기반 코딩 예시를 살펴보자.

테스트 코드에서 빈관련 설정 코드 넣을 때에는 필요로하는 객체 + TestConfig 라고 이름짓고 클래스르 만든다음.
주입받으려고하는 객체 빈 등록해주고, 이 설정 파일을 테스트할때 사용하려면 테스트 클래스에 @Import(객체 + TestConfig.class) 넣어주면 된다.


```java

import org.springframework.context.ApplicationEvent;

public class PostPublishedEvent extends ApplicationEvent {
    private final Post post;

    public PostPublishedEvent(Object source) {
        super(source);
        this.post = (Post) source;
    }

    public Post getPost() {
        return post;
    }
}

```

```java

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
@Import(PostRepositoryTestConfig.class)
public class PostRepositoryTest {
    @Autowired
    PostRepository postRepository;

    @Autowired
    ApplicationContext applicationContext;

    @Test
    public void event() {
        Post post = new Post();
        post.setTitle("event");
        PostPublishedEvent event = new PostPublishedEvent(post);

        applicationContext.publishEvent(event);
    }
}

```

```java

import org.springframework.context.ApplicationListener;

public class PostListener implements ApplicationListener<PostPublishedEvent> {
    @Override
    public void onApplicationEvent(PostPublishedEvent event) {
        System.out.println("-------------------------");
        System.out.println(event.getPost() + " is published!");
        System.out.println("-------------------------");
    }
}

```

```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class PostRepositoryTestConfig {

    @Bean
    public PostListener postListener() {
        return new PostListener();
    }
}

```

이러한 구조의 이벤트 코딩을 할수 있또록 스프링 데이터에서도 제공한다.

스프링 데이터 Common 10. QueryDSL 연동

사용 이유.

1. 조건문을 표현하는 방법이 굉장히 타입세이프 하다. 조건문들을 조합할 수 있다.

2. 쿼리 메서드의 단점은 함수의 이름이 굉장히 길어지는 단점이 있다. 가독성 감소.

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
    <artifactId>querydsldemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>querydsldemo</name>
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
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-apt</artifactId>
        </dependency>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated-sources/java</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```

```java

package me.whiteship.querydsldemo.account;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String firstName;

    private String lastName;

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

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

```

```javad

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.querydsl.QuerydslPredicateExecutor;

public interface AccountRepository extends JpaRepository<Account, Long>, QuerydslPredicateExecutor<Account> {

}


```

QuerydslPredicateExecutor 이 클래스에는 findOne, findAll 두가지 함수가 존재. findOne은 단일 건으로 Optional로 결과를 반환하고 findAll은 컬렉션으로 반환.
