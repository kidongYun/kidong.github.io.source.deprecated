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

