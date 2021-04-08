---
layout: post
title:  "스프링 부트 레디스 적용"
date:   2021-03-23 00:0054 +0900
categories: java
---

스프링 부트 프로젝트 생성 후 아래 의존성 추가.

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

docker로 레디스를 띄운다. redis 이미지를 가져오며. 해당 컨테이너의 이름을 redis_boot 로 명명. 이 redis 컨테이너는 호스트와 6379 포트로 통신한다.

```
docker run -p 6379:6379 --name redis_boot -d redis
```

레디스에 접속한다.

```
docker exec -i -t redis_boot redis-cli
```

Runner 클래스에 redis에 데이터 푸쉬하는 코드 작성.

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

keys * 명령어를 통해서 전체 키 목록 조회 가능.

get {key} 입력하게 되면 value 값이 반환됨.

remote dictionary server - 자료구조가 기본적으로 key-value 라는 이야기.

spring boot 에서 redis 커스터마이징 할때에는 application.properties 에서 적용 가능.
포트를 변경한다던지 등등

```
spring.redis.url=
spring.redis.port=
```

repository 를 활용해서 레디스를 사용해보자.

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
}
```

```java
public interface AccountRepository extends CrudRepository<Account, String> {}
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
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("keesun", "whiteship");
        values.set("springboot", "2.0");
        values.set("hello", "world");

        Account account = new Account();
        account.setEmail("kidong@gmail.com");
        account.setUsername("kidong");

        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getUsername());
        System.out.println(byId.get().getEmail());
    }
}

```

### 레디스 명령어

hget : hash 타입의 데이터에서 특정 필드를 가져옴.

hgetall : hash 타입의 데이터 모두를 가져옴. (컬렉션을 가져옴)


레디스 명령어들 

```
// foo 라는 키를 가진 데이터의 유효기간을 10초로 설정. 10초가 지나면 사라짐
// 캐시서버이기 때문에 유효기간 설정하는것은 중요.
expire foo 10
```

```
// 10초 후에 temp:key1 키를 가지고 "..." value 를 가지는 데이터가 사라진다.
set temp:key1 "it will delete after 10 seconds" ex 10
```

```
// 한번에 여러 개의 key-value 를 저장한다. 여기서는 a와 1의 묶음과, b와 2의 묶음을 저장한다.
mset a 1 b 2

// 한번에 여러 개의 key 데이터를 조회한다. 여기서 c는 없기 때문에 nil 값이 나온당.
mget a b c
```

```
//레디스에서 hash 구조는 key-subkey-value 구조를 가진다. 
hset key1 subkey1 "hello"
hset key1 subkey2 "world"

hget key1 subkey1
hkeys key1 -> 이 명령어는 레디스에 락이 걸리기 때문에 주의해야 한다.
```

```
// list 구조
// 리스트의 왼쪽에 a 값을 삽입
lpush gdhong:test "a"
// 리스트의 오른쪽에 b 값을 삽입.
rpush gdhong:test "b"
// 리스트의 왼쪽 부터 숫자 만큼 추출
lpop gdhong:test 2
// 리스트의 오른쪽 부터 숫자 만큼 추출
rpop gdhong:test 2
// 리스트를 주어진 숫자 범위만큼 검색.
lrange gdhong:test 0 4
```

```
// set 구조
// set 구조이고 gdhong:task 키에 "programming" value 삽입
sadd gdhong:task "programming"
// set 구조이고 gdhong:task 키에 "reading" value 삽입
sadd gdhong:task "reading"
// 위 명령어를 한번더 수행해도 삽입이 안됨. set이 중복이 안되는 구조이기 때문.

// set 구조의 키를 넣으면 해당 value 값들을 조회할 수 있다
smembers gdhong:task

// 만약 존재하는 value면 1을 반환, 아니면 0을 반환.
sismember gdhong:task reading - 1
sismember gdhong:task writing - 0

// 두 집합의 합집합을 구함 (중복은 제거된다는 의미)
sunion gdhong:task gdhong:task2

// 두 집합의 차집합을 구함
sdiff gdhong:task gdhong:task2

// 두 집합의 교집합을 구함
sinter gdhong:task gdhong:task2
```

```
// ordered set 집합이지만 오름차순으로 정렬되어 있음

// ordered set 컬렉션에 데이터를 넣음. 가중치 값이 추가로 들어간다.
zadd users:point 1 gdhong

// 조회.
zrange 
```


### SpringTemplate 을 활용한 레디스 명령어 처리