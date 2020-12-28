---
layout: post
title:  "Building Jpa with Querydsl"
date:   2020-10-26 11:38:54 +0900
categories: java
---

Gradle 프로젝트 기준으로 JPA에 QueryDSL을 연동하는 방법을 알아본다.

### build.gradle

QueryDSL은 QClass라는 객체를 자동으로 생성하고 이를 기반으로 타입 세이프한 쿼리를 작성할 수 있도록 돕는다. 이 QClass 객체들이 생성이 될 수 있도록 아래처럼 플러그인을 추가해줘야한다.

```

plugins {
    ...

    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    ...

}

```

아래는 QueryDSL 의존성 추가하는 코드이다.

```

dependencies {
    ...

    implementation 'com.querydsl:querydsl-jpa'

    ...
}

```

QueryDSL이 QClass를 만들 때 어느 경로에 생성할지와 같은 부가적인 설정 정보이다.

```

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}

test {
    useJUnitPlatform()
}

```

여기까지 추가했다면 Gradle 탭을 열고 Gradle build를 하자. 그러면 위에서 설정한 경로에 QClass 파일이 생성될 것이다.
확인해보자.

### QuerydslConfig.java

QueryDSL로 쿼리를 만들때 주로 사용하게 되는 객체는 JPAQueryFactory이다. 이 객체를 빈으로 등록하여 사용하자.

```java

@Configuration
public class QuerydslConfig {
    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() { return new JPAQueryFactory(entityManager); }
}

```

### Entity

QueryDSL이 정상 동작하는지 확인하기 위해 샘플 Entity를 만든다.

#### Cell.java

```java

@Slf4j
@Getter
@Setter
@ToString
@SuperBuilder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Cell {
    public enum Type { Objective, Plan, Todo }

    @Id
    @GeneratedValue
    protected Long id;

    protected LocalDateTime startDateTime;

    protected LocalDateTime endDateTime;

    protected String status;

    protected Type type;

    @ManyToOne(cascade = CascadeType.PERSIST)
    protected Member member;
}

```

### Repository

QueryDSL이 정상 동작하는지 확인하기 위해 샘플 Repository 만든다. JpaRepository를 상속받는 메인 Repository를 만들고 Custom Repository를 만든다. 그 후에 Custom Repository의 구현체를 만들고 여기서 QueryDSL을 사용한다.

#### CellRepository.java

```java

@Transactional
public interface CellRepository<T extends Cell> extends JpaRepository<T, Long>, CellRepositoryCustom<T> {
    List<T> findByType(Cell.Type type);
}

public interface CellRepositoryCustom<T extends Cell> {
}

@RequiredArgsConstructor
public class CellRepositoryCustomImpl<T extends Cell> implements CellRepositoryCustom<T> {
    private final JPAQueryFactory queryFactory;

    
}


```

You could use this repository when you need to create more complexible query. It's operated by querydsl.

#### ObjectiveQueryRepository.java

```java

@RequiredArgsConstructor
@Repository
public class ObjectiveQueryRepository {
    private final JPAQueryFactory queryFactory;

    public List<Objective> findByTitle(String title) {
        return queryFactory.selectFrom(objective)
                .where(objective.title.eq(title))
                .fetch();
    }
}

```

You can make a various queries using both method-query and querydsl.

### Test

Finally we can use the querydsl.

```java

@SpringBootTest
class ObjectiveRepositoryTest {

    @Autowired
    private ObjectiveRepository objectiveRepository;

    @Autowired
    private ObjectiveQueryRepository objectiveQueryRepository;

    @Test
    void save() {
        Objective obj = Objective.builder()
                .title("title")
                .priority(1)
                .status(1)
                .startDateTime(LocalDateTime.now())
                .endDateTime(DateTimeUtil.of(2021, 4, 30))
                .description("It's description")
                .build();

        objectiveRepository.save(obj);

        List<Objective> result = objectiveQueryRepository.findByTitle("title");

        System.out.println(result2);
    }
}

```