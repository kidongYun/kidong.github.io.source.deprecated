---
layout: post
title:  "The core of Spring Framework"
date:   2020-10-27 11:38:54 +0900
categories: spring
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


environment property. jvm.option 같은 것들 가져오는 방법 말하는 거 같다.

-Dvm.option 으로 줄수도있고
혹은 .properties 파일 안에다가 넣어둔 값도 아래처럼 가져올 수 있다.


```java
Environment environment = ctx.getEnvironment();
System.out.println(envrinment.getProperty("app.name"));
```

이렇게 바로 가져오는 방법도 있고.
아 이걸로 계정 정보를 넣어두는것도 괜찮았을거 같은데..

MessageSource
국제화 기능을 제공하는 인터페이스.

```properties
greeting=안녕, {0}
```

```java

@Autowired
ApplicationContext applicationContext;

@Autowired
MessageSource messageSource;

@Override
public void run(ApplicationArguments args) throws Exception {
    System.out.println(messageSource);

    System.out.println(messageSource.getMessage("greeting"), Locale.KOREA);

        System.out.println(messageSource.getMessage("greeting"), Locale.getDefault());
}

```

message.properties
messages_ko_KR.properties

이와 같이 파일이름을 해서 만들어줘야함.

그러면 mesaageSourcerk message로 시작하는 properties 파일들을 자동적으로 읽어서 가져온다.

빈등록해서도 사용가능

```java

@Bean
public MessageSource messageSource() {
    var messageSource = new ReloadableResourceBundleMessageSource();
    messageSource.setBasename("classpath:/messages");
    messageSource.setDefaultEncoding("UTF-8");
    return messageSource;
}
```

릴로딩 기능이 있다.

-> 빌드만 해주면 메시지 번들이 자동으로 빌드대서 변경된 내용이 적용된다.

내가 쓰려는 타입으로 선언해서 쓰는것이 좋다. 상위 크래스를 선언하는 것보다. 왜냐하면 어떤 용도로 쓰려는것인지 보다더 가독성이 드러나기 떄문.


```java
ApplicationEventPublisher
이벤트 기반의 코딩을 할떄 사용??

public class MyEvent extends ApplicationEvent {
    private int data;

    public MyEvent(Object source){
        super(source);
    }

    public MyEvent(Object source, int data) {
        super(source);
        this.data = data;
    }

    public int getData() {
        return this.data;
    }
}
```

여기부분이 리스너다.

```java

@Component
public class MyEventHandler implements ApplicationListener<MyEvent> {
    @Override
    public void onApplicationEvent(MyEvent event) {
        System.out.println("이벤트 받았다. 데이터는 " + event.getData());
    }
}
```

```java

@Autowired
ApplicationEventPoblisher publishEvent;

@Override
public void run(ApplicationArguments args) throws Exception {
    publishEvent.publishEvent(new MyEvent(this, 100));
}

```

스프링 부트 설정하는 부분에서 이벤트를 등록하는 코드를 작성.

이벤트는 ApplicationEvent 를 상속 받아서 미리 구현해두고.

이 등록된 이벤트는 등록되어져 있는 빈들 중에 적절한 리스너를 찾아서 호출한다.

리스너는 위에 구현해놔씀

4.2 이전 버전 방법이고.

이후에는 이렇게 사용하지 않음. 스프링은 자기의 프레임워크가 비지니스 코드들에 들어가지 않기를 원함 비침투성을 원함 그런데 4.2 버전 이전에는 ApplicationEvent 등을 상속받는다거나 하는것들이 모두 스프링프레임워크에 의존되는 코드들 넣어야 했음. 이후에는 이제 안그래도 된다고 함.

어케 하느냐

리스너에 해당하는 코드에 @EventListener 어노테이션 달기만 하면 됨.

Event 클래스는 POJO 클래스가 되었고 이를 @EventListener 가 리스너로 받는다

```java

@Component
public class MyEventHandler(MyEvent event) {
    
    @EventListener
    @Order(Ordered.HIGHEST_PRECEDENCE + 2)
    public void handle(MyEvent event) {
        System.out.println(Thread.currentThread().toString);
        system.out.println("이벤트 받았다. 데이터는 " + event.getData());
    }
}

```

비동기적으로 쓰고시으면 @Async 붙여라. + 그리고 스프링 부트 설정 파일 ...Application 이 클래스에 @EnableAsync 이걸 붙이면 각각의 쓰레드를 호출해서 비동기적으로(비순차적으로) 실행된다.

ApplicationContext의 기능 ResourceLoader.

말 그대로 resource를 로딩해주는 역할을 하는데 이를 applicationcontext가 상속받고 있다.

```java

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        resourceLoader.getResource("classpath:test.txt");
        System.out.println(resource.exists());
    }
}

```

resource 폴더 안에있는 것들이 
빌드되었을때 target > classes 폴더 안에 들어아깄을거임.

classpath가 저길 바라보는 것인가.

 ApplicationContext는 BeanFactory + Message... + Event + Resource ... 등등 많은 기능을 내포.

 Resource 는 java.net.URL 이 객체를 추상화시켜 만든것.

 기존의 것이 클래스패스를 기준으로 가져오는 경우가 없었고..
 HTTP, FTP emd prefix 에 대한 기능도 없었고.

 방법을 하나로 통일. getResource()

 Resource 객체가 스프링 내부적으로도 많이 사용된다.

 xml로 스프링 설정 했을 때에도 ClassPathXmlApplicationContext() 이 함수로 가져오는데.
 여기 안에 파라미터로 들어가는 문자열이 Resource 객체로 된다.

 이 Resource를 구현한 구현체들은 굉장히 만흥ㄴ데.

 이중 UrlResource, ClassPathResource, FileSystemResource, ServletContextResource 이렇게 있따.

 classpath:
 file:/// 등과 같이 접두어를 사용해서 해당 파일이 어디에서 오는지 명시를 해주는 것이 더 좋다.

 getResource 할때 location의 접두어에 따라서 실제로 생성된 sub-class가 다르다.

 classpath 접두어를 지우면 default로 ServletContextResource 를 사용한다.
이거는 웹 애플리케이션 루트에서 상대 경로로 리소스를 찾는다.

applicationContext를 어떤것을 쓰냐에 따라서 저 경로가 달라지기 떄문에 명시적으로 경로를 적어주는 것이 좋다.

Validation

Validator. 웹 MVC에서 많이 사용하지만 사실 이것만을 위해서 만들어진 인터페이스는 아니다.

Bean Valdidation.
-> Java EE 표준 스펙.
-> NotEmpty, NotNull, NotBlank.... 등등

Validator를 사용하기 위해서는 2가지 동작을 구현해야함.

support()
validate()
이 두개.

```java

public class Event {
    Integer id;
    String title;
}

```

```java

public class EventValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Event.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "notempty", "default message")
    }
}

```

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        Event event = new Event();
        EventValidator = eventValidator = new EventValidator();
        Errors errors = new BeanPropertyBindingResult(event, "")

        eventValidator.validate(event, errors);

        System.out.println(errors.hasErrors());

        errors.getAllErrors().forEach(e -> {
            System.out.println("===== error code =====");
            Arrays.stream(e.getCodes()).forEach(System.out.::println);
            System.out.println(getDefaultMessage());
        });
    }
}

```

이렇게 Validator를 직접 인스턴스 생성해서 하는 방법은 정말 원시적인 방식.

ValidationUtils 를 사용하지 않고 직접 조건을 걸수 있따.

```java

Event event = (Event) target;
if(event.getTitle() == null) {
    errors.reject()
}

```

```java

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    Validator validator

    ....

}

```

```java

public class Event {
    Integer id;

    @NotEmpty
    String title;

    @NotNull @Min(0)
    Integer limit;
}
```

데이터 바인딩 -> 컨트롤러 등에서 데이터를 받아올때 이를 string type이 아니고 도메인으로 바로 받을수 있게끔 하는 기능

```java

public class Event {
    private Integer id;
    private String title;
}

@RestController
public class EventController {

    @InitBinder
    public void init(WebDataBinder webDataBinder) {
        webDataBinder.registerCustomEditor(Event.class, new EventEditor());
    }

    @GetMapping("/event.{event}")
    public String getEvent(@PathVariable Event event) {
        System.out.println(event);
        return event.getId().toString();
    }
}

@RunWith(Springrunner.class)
@WebMvcTest
public class EventControllerTest {
    @Autowired
    MockMvc movkMvc;

    @Test
    public void getTest() {
        mockMvc.perform(get("/event/1"))
        .andExpect(status().isOk())
        .andExpect(content().string("1"));   
    }
}

```

{event} 에 해당하는 1의 값이 string 형태인데 이걸 Event 타입으로 변경하지 못하기 때문에 오류가 날거임.

이럴때 사용하는게 PropertyEditor 를 사용해서 적용할 수 있음.

```java

public class EventEditor extends PropertyEditorSupport {
    @Override
    public String getAsText() {
        Event event = (Event)getValue()

        return event.getId().toString();
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        setValue(new Event(Integer.parseInt(text)));
    }
}

```
getValue(), setValue() 가 서로다른 쓰레드에게 공유가 되기 때문에 Stateful하다. 상태정보를 저장하고 있다. Thread-safe 하지 않다.
여러 쓰레드에서 공유해서 사용하면 안된다 -> 빈으로 등록하면 안된다. (일반적으로 싱글톤이기 때문)
그냥 빈이 아닌 Thread Scope 으로 생성해서 사용하면 가능. 근데 그냥 빈으로 등록 안하는 것을 추천함.



구현체를 직접 구현하는 경우에는 구현해야할 메서드들이 매우 많아지기 때문에 일반적으로 이러한 interface를 구현해둔 디폴트 클래스가 있다.
이걸 상속받고 필요한 메서드들만 오버라이딩 해서 구현하는게 일반적이다.

PropertyEditor는 구현하기도 너무 거추장하고
 Thread-safe 하지 않고 
 Object -> String 으로 변환이 가능하다는 단점 때문에 새로 추가된 것이 있는데 그게 Converter와 Formatter



Converter
S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기.
상태 정보를 없앴음 == Stateless == Thread-safe


```java

public class EventConverter implements {
    /* Convert from String to Event */
    public static class StringToEventConverter implements Converter<String, Event> {
        @Ovevrride
        public Event convert(String source) {
            return new Event(Integer.parseInt(source));
        }
    }

    public static class EventToStringConverter implements Converter<Event, String> {
        @Override
        public String convert(Event source) {
            return source.getId().toString();
        }
    }
}

```

Bean 으로 등록해도 되며 보통 ConverterRegistry 에 등록한다.


BOOT가 아닌경우
```java

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new EventConverter.StringToEventConverter());
    }
}

```

Formatter

```java

public class EventFormatter implements Formatter<Event> {
    @Override
    public Event parse(String, text, Locale locale) throws ParseException {
        return new Event(Integer.parseInt(text));
    }

    @Override
    public String print(Event object, Locale locale) {
        object...;
    }
}

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new EventFormatter());
    }
}

```

타입은 모두다 ConversionService.
WebConversionService extends DefaultFormattingConersionService;
Web... 요거가 스프링 부트에서 제공해주는거.

converter, formatter 사용을 위해서 스프링 부트에서는 webconfig를 새로 추가해서 등록할 필요가 없다.
자동으로 Conversion service에 빈등록 시켜준다. 

@WebMvcTest()
-> 계층형 테스트, 웹과 관련된것만 Bean 등록, 보통 컨트롤러만 빈등록이 됨, 컨버터나, 포매터는 등록이 안될 가능성이 있다.

```java
@WebMvcTest({EventFormatter.class, EventController.class})
```
저렇게 {} 안에다가 추가적으로 객체를 넣어주면 특정 객체도 수동적으로 빈 등록이 가능하다.


등록되어있는 컨버터들을 전부 보는 방법.
```java
System.out.println(conversionService);
```

SpEL (Sprng Expression Language)

```java

/* application.properties */
my.value = 100

/* bean sample */
@Getter
@Setter
@Component
public class Sample {
    private int data = 200;
}


@Component
public class AppRunner implements ApplicationRunner {
    @Value("{#{1 + 1}}")
    int value;

    @Value("#{'hello' + 'world'}")
    String greeting;

    /* 표현식을 사용하는 방법 */
    @Value("#{1 eq 1}")
    boolean trueorFalse;

    /* 프로퍼티 접근 */
    @Value("${my.value}")
    int myValue;

    /* 표현식 안에서도 프로퍼티 접근이 가능함.*/
    @Value("#{${my.value} eq 100}")
    boolean isMyValue100;

    /* 문자열이 그대로 들어간다. */
    @Value("#{'spring'}")
    String spring;

    @Value("#{sample.data}")
    int sampleData;

    /* 특정 조건에 따라 선별적으로 빈등록을 해주는 어노테이션. */
    @ConditionalOnExpression 

    /* 이 annotation을 사용해서 쿼리 만들때 파라미터 전달할 때에도 SpEL 사용. */
    @Query


    @Override
    public void run(ApplicationArguments args) throws Exception {

    }
}
```

가장 심플한 SpEL 사용법.

SpEL 어떻게 동작하는가.

```java

ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("2 + 100");
Integer value = expression.getValue(Integer.class);
System.out.println(value);

```

AOP 개념
-> 흩어진 Aspect를 모듈화 할수 있는 프로그래밍 기법.

concerns -> 비슷한 형태의 코드

대표적인 예제가 transaction -> transaction 처리를 할때 set auto commit .... commit rollback.
기존의 서비스 코드를 하나로 묶는 것.

transaction 처리!!!! 요거 함 봐야될거 같은데. 중요한거같은데.

이러한 유사한 코드들 concern 들을 Aspect 라는 곳에 모아둬서 처리한다.

해야할일과 어디에 적용해야하는지를 묶어서 저장해둔다. 이게 AOP 처리 기법.

용어가 좀 어려워서 처음에 포기하는데 실제로는 어렵지 않다.

Aspect -> 묶은 모듈
    이 모듈에 Advice, Pointcut 두가지가 들어가는데

    Advice -> Aspect로 관리되는 코드
    Pointcut -> 어디에 적용해야하는지

Target -> 적용이 되는 대상

JoinPoint -> 메서드 실행시점. 쓰레드가 분기되고 합쳐지는 것처럼. 합쳐지는 지점을 의미.

CrossCut -> 흩어져있따.

AOP 적용 방법
    - 컴파일 타임 => 바이트코드를 만들때 수정된 버전으로 만든다.
        -> It doesn't have any limit when it is running.
    - 로드 타임 => jvm 로드타임 위빙. 낑겨서 넣는다
        -> you should set the load time weaver. and it would occur a little of load.
    - 런 타임 => 스프링 AOP가 사용하는 방법. -> A라는 클래스의 빈을 만들때 프록시 빈을 만든다.. 프록시 기반의 AOP.
        -> 가장 유동적이나 부하가 클수 있음. 최초에 빈을 생성할때 약간의 성능 이슈가 있을수 있으나 로드 타임과 거의 유사하다고 봄.
        -> 런 타임에서 적용하기 때문에 별도의 설정등이 적다.

AspectJ (컴파일, 로드타임에서도 제공)
-> 매우 많은 JoinPoint 등을 제공 

Spring AOP (런타임으로 제공)
-> 제한된 환경으로 제공.

Spring AOP dependency는 Spring-core, Spring-context에 없다 그래서 새로 추가해야함.

Spring AOP 특징
-> 프록시 기반의 AOP
-> 스프링 빈에만 AOP를 적용

client -> subject(interface) -> proxy, read subject
프록시 패턴  : 기존 코드 변경없이 접근 제어 또는 부가 기능 추가.

```java

public interface EventService {
    public void createEvent();
    public void publishEvent();
    public void deleteEvent();
}

public class SimpleEventService implements EventService {
    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Created an event");

        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Published an event");

        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        System.out.println("Deleted an event");
    }
}

public class AppRunner implements ApplicationRunner {
    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throw Exception {
        eventService.createEvent();
        eventService.publishEvent();
        eventService.deleteEvent();
    }
}

```

위의 코드는 성능 측정을 위해 기존 코드에 추가하고 있음.
이런거를 하고싶지 않다! 이럴때 프록시 패턴을 사용!!

성능을 측정하는 코드 같은것들은 사실 메인의 기능이 아님. 이런것들을 프록시에 넣어두면
기존코드를 수정하지 않으면서 이러한 새로운 코드를 추가할 수 있다.

한 인터페이스에 모두가 동일하게 사용하는 기능들이 있을경우.
이런걸 프록시 패턴을 사용해서 처리하면 좋은가?? 이거 좋을거 같은데!!?!?!?

이거 필요했는데

```java


public interface EventService {
    public void createEvent();
    public void publishEvent();
    public void deleteEvent();
}


@Primary
@Service
public class ProxySimpleEventService implements EventService {
    
    @Autowired
    SimpleEventService simpleEventService;
    
    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println(System.currentTimeMillis() - begin);
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
    }
}

public class SimpleEventService implements EventService {
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Created an event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Published an event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Deleted an event");
    }
}

public class AppRunner implements ApplicationRunner {
    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throw Exception {
        eventService.createEvent();
        eventService.publishEvent();
        eventService.deleteEvent();
    }
}

```

ProxyService 이거를 만드는데 드는 비용이 꽤 크다.
그리고 기능이 결국 비슷한걸 적용하기위해 Proxy를 만들었으니 이걸 AOP 로 사용??
-> 프록시 객체를 동적(런타임 시점에)으로 만드는 방법이있다.
-> 어떤 객체를 감싸는 프록시 객체를 만드는 거(프록시 객체 자체는 굉장히 심플)
-> 

AbstractAutoProxyCreator 를 활용해서 SimpleEventService 를 감싸는 프록시 객체를 만들어서 빈 등록해준다.
AbstractAutoProxyCreator는 BeanPostProcessor 라이프사이클을 구현한 녀석.


```
dependency 추가
spring-boot-starter-aop
```

```java

@Component
@Aspect
public class PerfAspect {

    /** 요거가 Advice 이다. */
    /** Around 안에있는 것이 Pointcut. */
    @Around("execution(* me.whiteship..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint pjp) throw Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }
}

```

Advice : 해야할 일
PointCut : 적용되는 지점

AOP 정말 필요한 기능이였다. 프록시 패턴을 활용해서 AOP를 구현하는것 같다.
-> AOP는 보통 annotation으로 활용해서 쓰는게 좋다.

```java

/** 이 어노테이션 정보를 어디까지 유지할 것인가. CLASS 라면 CLASS 까지 유지한다는 의미*/
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerLogging {

}


@Component
@Aspect
public class PerfAspect {

    /** 요거가 Advice 이다. */
    /** Around 안에있는 것이 Pointcut. */
    @Around("@annotation(PerLogging)")
    public Object logPerf(ProceedingJoinPoint pjp) throw Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }
}

public class SimpleEventService implements EventService {
    @PerLogging
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Created an event");
    }

    @PerLogging
    @Override
    public void publishEvent() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Published an event");
    }

    @Override
    public void deleteEvent() {
        System.out.println("Deleted an event");
    }
}

```

위처럼 annotation 기반으로도 Pointcut을 설정할 수 있다.

```java

@Before("bean(simpleEventService)")
public void hello() {
    System.out.println("hello");
}

```
이렇게 빈으로도 등록이 가능. @Before 는 해당 메서드가 실행되기 이전에만 실행되게금.

```
어드바이스 정의
@Before
@AfterReturning
@AfterThrowing
@Around
```

```
포인트컷 정의
@Pointcut
@annotation
bean
execution
....
```