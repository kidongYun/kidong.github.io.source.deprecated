---
layout: post
title:  "The core of Spring Framework"
date:   2020-10-27 11:38:54 +0900
categories: java
---

### Inversion of Control (IOC)

IOC 기능을 제공하는 컨테이너 안에는 빈들을 저장한다. 이 빈들을 DI 하여 바로바로 사용할 수 있도록 한다.

초기에는 xml 기반으로 Spring 기본 설정을 하였으나 이후에는 annotation 기반으로 Spring 기본 설정을 함.

스프링 IoC 컨테이너.
    - BeanFactory.
    - 애플리케이션 컴포넌트들의 중앙 저장소.
    - 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.

빈
    - 스프링 IoC 컨테이너가 관리하는 객체.
    - 스프링 IoC 컨테이너가 관리하지 않는다면 이는 빈이 아니다. 단순히 자바 객체이다.
    - @Repository -> auto-scan -> 빈 생성.

스프링에서 의존성 주입(DI)를 받으려면 빈으로 관리되어야 한다.
    -> 빈으로 관리하고 싶을때에는 해당 객체가 싱글톤 패턴을 준수해도 되는지를 먼저 고려하면 좋다. 기본적으로 빈들은 싱글톤 패턴으로 생성되기 때문. (<-> 프로토타입 패턴)

    -> 이미 생성된 빈들을 가지고 활용하기 때문에 메모리 절약, 런타임시 비용 절감에 대한 장점을 가진다. (싱글톤의 장점)

    -> 스프링 IoC 컨테이너에 등록된 빈은 스프링에서 제공하는 라이프사이클로 관리할 수 있다.
    
    라이프사이클 콜백 예시)
    @PostConstruct
    -> 빈이 생성된 이후에 위 annotataion이 붙은 함수가 호출된다.

    우리가 직접 IoC를 만들수도 있지만 이미 Spring에서 제공하는 IoC의 기능이 충분히 범용적이고 효율적이기 때문에 이를 사용하는게 더 낫다.

    의존성 문제.

    BookService 객체가 현재 BookRepository 객체를 내부에서 사용하고 있다. 이말은 즉슨 BookService가 BookRepository 에 대한 의존성을 가지고 있다고 볼수 있다. 이렇게 의존성을 가진 BookService를 테스트하기 위해서는 BookRepository가 필요하기 때문에 단위테스트를 하기가 어려워진다.

    BookService 내부에서 이렇게 사용되어지는 BookRepository를 자체적으로 생성한다면. 해당 객체의 의존성 주입은 BookService에 항상 의존된다. 이런 문제를 해결하기 위해서 DI 작업은 외부에서 주입될수 있는 구조로 설계 해야한다.


@Mock
-> 가짜 객체를 생성한다.

의존성 주입을 외부로 빼둔다면 내부에서 사용되는 객체를 가짜로 생성하든, 실제로 생성하는 그때그때 필요한데로 구성해서 주입해줄수 있다.

ApplicationContext
-> BeanFactory 중 하나


빈주입하기.

빈주입을 하려면 스프링 IoC 컨테이너가 있어야겠쥬.

(1) applicationContext.xml
빈 주입을 관리를 xml을 활용해서 하는 방법. 이건 진짜 old 스타일. 안좋은점이 빈 주입을 위한 작업이 굉장히 많다. xml에 넣어줘야하는 코드들이 상당하다. 그리고 관리 포인트가 자바가 아니기 때문에 별도로 존재하는 파일의 느낌이 듬.

autowired 는 기본적으로 타입과, 이름으로 빈을 추론.

한 빈 안에 의존성을 가지고있다면 해당 의존성을 주입해주기 위해서 xml 에서 <bean/> 태그 안에 <property/> 태그를 추가해야함.<property/> 태그 속성 중 ref 가 있는데 여기에 주입하려고 하는 빈의 id 값을 넣어주면 됨.


-> 빈을 일일히 작성해줘야 하는것이 정말로 번거롭다.

그래서 등장한 것이 component-scan, auto-scan.

xml 파일안에 <context: .../> 와 같은 태그를 사용해서 특정 패키지에 존재하는 빈들을 찾아서 빈으로 등록된다.

빈등록과 의존성 주입은 다르다.

auto-scan을 해두고 빈으로 등록하고 싶은 클래스들에 @service, @component, @repository 등등의 어노테이션을 박아두면 자동으로 빈 등록이 된다.

auto-scan으로 설정한 패키지와 어노테이션을 등록한 클래스의 위치가 맞아야만 등록된다

-> 그다음은 xml 기반이아닌 java 기반으로 bean 등록하기.

@Bean 어노테이션을 활용해서 빈 등록.
빈 등록할때에도 보이는 것이 해당 빈이 가지고 있는 의존성이 외부에서 주입될 수 있도록 코드를 왜 짜야하는지 알수 있을 것.

setter를 활용해서 직접 의존성을 주입하는 방법과 달리 @autowired 를 사용하면 IoC 컨테이너를 먼저 확인을 하고 해당 빈과 합당한 빈을 찾을 수 있는 경우 바로 그 빈을 주입한다.

-> java로 bean 등록 할때에도 @ComponentScan() 을 사용하면 적용된 패키지 안에있는 클래스들은 모두 자동 빈 등록된다.

@SpringBootApplication을 사용하면 자동적으로 ApplicationContext를 생성해준다.

스프링부트를 실행하면 생성되어 있는 Application.java 파일이 사실상 어노테이션으로 되어있지만 스프링 설정파일이라고 생각하면된다. 어노테이션들을 잘 확인해보면 알수 있다.


ApplicationContext
    - ClassPathXmlApplicationContext (XML)
    - AnnotationConfigApplicationContext (JAVA)


@Autowired

```java

@Service
public class BookService {
    BookRepository bookRepository;

    @Autowired
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

생성자 주입을 하는 코드이다. 생성자에서 @Autowired 선언을 해두면 해당 파라미터 객체들은 빈에서 가져오게 된다. 

```java

@Service
public class BookService {
    
    BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}

```

이번에는 setter 에서 주입을 시도한다.
실제로 setter는 무조건 호출되는 코드가 아니기 때문에 BookRepository 빈이 없어도 BookService 객체가 생성되는 것은 맞는데. @Autowired 는 무조건 호출되기 때문에 오류가 난다. 만약 호출하기 싫다면 @Autowired(required = false) 로 어노테이션 하면 된다.

생성자 주입에 비해 setter 주입이 좀더 자유롭다. BookService 를 먼저 만들고 추후에 의존성을 주입할 수 있으니까.

필드 주입은 이미 알고있는 그대로.




해당 타입의 빈이 여러개인 경우에는 명시를 해주지 않으면 스프링이 어떤 빈인지 모르기 때문에 주입해줄수 없다.

이럴땐 @Primary 사용하면 이렇게 여러개인 경우 @Primary를 우선적으로 주입한다.

혹은 중복되는 빈들을 모두 받을 수도 있다.

```java

@Autowired
List<BookRepository> bookRepositories;

```

ApplicationRunner 가 뭐지.

@Autowired
-> type 먼저 확인 후 그다음에 이름을 확인하긴 한다. 그러나 비추 왜냐하면 타입세이프하지 않으니. 아래 코드와 같음.

```java

@Autowired
BookRepository myBookRepository;

```

BeanPostProcessor
-> 라이프사이클인데 빈의 인스턴스를 만들고 initialization 하기전에 부가적인 작업을 하기위한 콜백.

initialization 이란 뭐냐

@PostConstruct
-> 빈주입 이후에 발생하는 콜백.

빈 등록할때 component scan 에 클래스 정보를 너어줄때 두가지 방법이 있는데
basePackages -> String 값을 받고 
basePackagesClass -> 요거는 클래스를 받기떄문에 보다더 타입 세이프하다.

스프링부트의 경우 Application.java 에 config 관련 어노테이션이 있는데 component-scan도 이 패키지 하위만 가능.

따라서 저 파일이 없는 외부의 패키지의 경우는 자동으로 scane되지 않는다. 따로 추가해줘야한다.

빈주입이 안될떼는 컴포넌트스캔을 잘 살펴보라.

@Filter 어노테이션은. 빈생성을 할때 요구조건에 따라 추가 제거를 할 수 있따.

component-scan에 중요한 것은 스캔의 범위와. 필터링되어있는 빈들이 어떤거인지를 확인하는것.

스프링의 빈 등록은 스프링 프로젝트가 처음에 시작될때 모두가 빈을 등록하기때문에 구동시에는 좀 느려질수가 있따.

이런거에 민감한 프로젝트는 빈 등록을 스프링 부트가 뜨는 시점에 registerBean() 함수를 활용해서 생성 시점을 변경할 수 있나보다.

```java
public static void main(String[] args) {
    var app = new SpringApplication(Demospring51Application.class);

    app.addInitializers((ApplicationContextInitializer<GenericApplicationContext>) ctx -> {
        ctx.registerBean(MyService.class);
    });

    app.run(args);
}
```

java10 이후부터는 var 라는 로컬 변수명을 가질 수 있다. -> 타입을 자동 추론하는 건가..?


@ComponentScan은 BeanFactoryPostProcessor 라이프사이클에서 빈으로 등록한다.

@ComponentScan이 빈등록 하는 대상은 @Component 어노테이션을 선언한 클래스이다.

@Repository, @Service, @Controller, @Configuration 들도 @Component 어노테이션을 포함하고 있다.

Function을 활용한 빈등록 (위에 언급한 방법)으로는 모든 빈들을 생성하기보다는 특별하게 @Bean 을 사용해서 등록하는 빈들만 쓰는게 나을듯.

빈의 스코프

싱글톤 -> 모든 프로젝트 전반에 걸쳐서 해당 인스턴스(빈)이 오직 한개인 형태.

스프링의 기본 빈 생성방식은 싱글톤이다. 그래서 @Component 만을 붙였을 경우 이 클래스는 하나의 인스턴스만 생성된다.

대부분의 경우 싱글톤 스코프를 사용하는데 그렇지 않은 경우 프로토타입 스코프를 사용한다.

프로토타입 스코프 -> 매 빈이 주입될 떄 새로운 인스턴스로 가져오는 것.

```java

@Component @Scope("prototype")
public class Proto {

}

```

문제는 싱글톤 스코프의 빈이 프로토타입 스코프의 빈을 의존할고 있을 때가 이슈가 있다.

왜냐면 싱글톤 스코프의 빈은 한번만 생성되기 때문에 매번 프로토타입의 스코프의 빈을 새로 생성해주지 않는다 즉 변경되지 않는다.

이를 해결하기 방법.

(1) proxyMode 를 사용한다.

```

@Component @Scope(value = "prototype", proxyType = ScopedProxyMode.TARGET_CLASS)
public class Proto {

}

```

Proxy로 감싸고. Proxy 빈을 사용하도록 한다. 싱글톤 스코프를 가진 빈이 프로토타입 스코프를 가진 빈을 의존하고 있을 때 직접 참조를 하게 되면 프로토타입 스코프의 빈이 변경이 될 수 없기 때문에 프록시라는 걸로 한번 감싸고 이 프록시의 빈이 변경되지 않도록 하고. 그 안에 있는 실제 프로토타입 빈은 변경될 수 있도록 한다.

Proxy 선언을 하면 이 프록시 객체가 전혀 새로운 객체가 되는것은 아니고 감싸지는 클래스와 동일하다.

롱런하는 빈이 짧은 생명 주기를 가진 빈을 의존할 때에는 이 Scope를 고려해야한다.


프로파일.

각 서버(테스트, 스테이징, 프로덕트 등)에 종속적으로 필요한 빈들을 추가하기 위한 기능?

```java
Environment environment = ctx.getEnvironment();

/* 프로파일 검색 */
System.out.println(Arrays.tostring(environment.getActiveProfiles()));

/* 기본적으로 적용되는 프로파일. */
System.out.println(Arrays.toString(environment.getDefaultProfiles()))

```

일반적으로 생성한 빈들은 모두 우리가 특정 프로파일을 언급하지 않았기 때문에 이 defaultProfiles 에 들어갔을 것이다.

```java

@Configuration
@Profile("test")
public class TestConfiguration {
    @Bean
    public BookRepository bookReposigory() {
        return new TestBookRepository();
    }
}

```

test라는 프로파일로 스프링을 실행하지 않으면 위 빈 등록하는 코드가 실행되지 않는다. 이를 통해 서버 단위로 구분되어지는 빈 등록이 가능하다.

프로파일 설정.
(1)
intellij setting -> active profiles

(2)
VM options
-Dspring.profiles.active="test"

2번 방법이 나은듯. 왜냐면 결국 서버에서 jar로 스프링 실행할 때 jvm option 줘야하기 때문.

```java

@Repository
@Profile("test")
public class TestBookRepository implements BookRepository {

}

```

@Configuration 쪽 뿐만아니라 Component-Scan 되어지는 빈들에게도 @Profile 활용할 수 있다.

@Profile을 적용하지 않은 경우는 -> @Profile("default") 로 생각하면 된다.

