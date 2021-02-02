---
layout: post
title:  "Spring data redis"
date:   2021-02-02 00:0054 +0900
categories: java
---

의존성 추가

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

도커 기반 Redis 설치

```
docker run -p 6379:6379 --name redis_boot -d redis
```

redis 프로그램으로 진입

```
docker exec -i -t redis_boot redis-cli
```

아래 명령어로 키 기반 조회 가능

```
key *
```

각 키에 해당하는 값을 알고 싶으면 아래 명령어 사용 가능

```
get keesun
```

기존에는 Redis에서 제공하는 client 라이브러리가 있었는데. 스프링 데이터에서 이 Redis를 감싸고 그 구현체를 제공한다.
그 구현체가 StringRedisTemplate, RedisTemplate

```java
@Component
public class RedisRunner implements ApplicationRunner {
    @Autowired
    StringRedisTemplate redisTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("keesun", "whiteship");
        values.set("springboot", "2.0");
        values.set("hello", "world");
    }
}
```

Redis 커스터마이징
application.properties 여기에서 spring.redis.* 이쪽 변경해주면 된다.
Redis는 기본적으로 6379 포트를 사용한다. 그렇기 때문에 docker와 Spring 모두 6379 포트가 기본값이 되어있다.

Repository를 만들어서 사용하는 방법

```java
@RedisHash("accounts")
public class Account {
    @Id
    private String id;

    private String username;

    private String email;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
```

```java
public interface AccountRepository extends CrudRepository<Account, String> {
}

```

```java
@Component
public class RedisRunner implements ApplicationRunner {
    @Autowired
    StringRedisTemplate redisTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setEmail("whiteship@email.com");
        account.setUsername("keesun");

        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getUsername());
        System.out.println(byId.get().getEmail());
    }
}
```

hash들은 get hello 이런식으로 못가져 온다.  hash get 으로 가져와야 한다.

```
hget accounts:1b27ba23-e840-4a6a-b6c4-41b036dabfbd email
hgetall accounts:1b27ba23-e840-4a6a-b6c4-41b036dabfbd
```