---
layout: post
title:  "Building Jpa with Querydsl"
date:   2020-10-26 11:38:54 +0900
categories: java
---

### build.gradle

add the plugin related to querydsl.

```

plugins {
    ...

    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    ...

}

```

add the dependency of querydsl for jpa.

```

dependencies {
    ...

    implementation 'com.querydsl:querydsl-jpa'

    ...
}

```

and we should set the directory path for putting on the QClass files, which is created when you build querydsl.
Those objects are needed when you consist the querydsl.

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

Let's open the gradle window and build it. then you can see the QClass files in the root directory which is set at the above.

### QuerydslConfig.java

The main object for creating querydsl is JPAQueryFactory. we gotta set this object as a bean.

```java

@Configuration
public class QuerydslConfig {
    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() { return new JPAQueryFactory(entityManager); }
}

```

After this, We could access this using the bean annotation like @Autowired, @Resource, @RequiredArgsConstructor and so on

It's the end of the basic setting for using querydsl. if you already have the entity, repository then you just create new custom repository with querydsl.

### Entity

It's my example source code in my project. It could be little difficult bcs this code has my business thing. I mean it is not simple entity.

#### Cell.java

```java

@Getter
@Setter
@ToString
@Component
@MappedSuperclass
public class Cell {
    protected String type;
    protected LocalDateTime startDateTime;
    protected LocalDateTime endDateTime;
}

```

#### Subject.java

```java

@Getter
@Setter
@ToString(callSuper = true)
@Component
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Subject extends Cell {
    @Id
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "SEQ_GEN")
    @SequenceGenerator(name = "SEQ_GEN", sequenceName = "ID_SEQ")
    @Column(name = "ID")
    protected long id;

    @Column(name = "STATUS")
    protected int status;
}

```

#### Objective.java

```java

@Slf4j
@Getter
@Setter
@ToString(callSuper = true)
@Component
@Entity
@Table(name = "OBJECTIVE")
@AttributeOverrides({
        @AttributeOverride(name = "startDateTime", column = @Column(name = "OBJ_START_DATETIME")),
        @AttributeOverride(name = "endDateTime", column = @Column(name = "OBJ_END_DATETIME")),
        @AttributeOverride(name = "status", column = @Column(name = "OBJ_STATUS"))
})
public class Objective extends Subject {
    @Column(name = "OBJ_TITLE")
    private String title;

    @Column(name = "OBJ_DESCRIPTION")
    private String description;

    @Column(name = "OBJ_PRIORITY")
    private int priority;
}

```

### Repository

First one is basic repository is offered from Spring Data JPA. You can use method query using this.

#### ObjectiveRepository.java

```java

public interface ObjectiveRepository extends SubjectRepository<Objective, Long> {}

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