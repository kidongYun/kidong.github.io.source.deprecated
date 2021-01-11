---
layout: post
title:  "테스트 코드 작성시 stub 객체에 대한 고찰"
date:   2021-01-06 20:00:00 +0900
categories: java
---

테스트 코드를 작성하다보면 서로 간에 의존성을 가지고 있어서 테스트를 하기 어려운 상황들을 직면하게 된다. 이를 해결하기 위한 나의 생각들을 남겨두려고 한다. 이 글은 저자의 주관적인 생각이 포함되어 있음으로 맹목적인 신뢰는 지양하기 바란다.

### 외부 리소스 코드의 분리

테스트하려는 코드가 의존성을 가지고 있어서 테스트하기가 어렵다면, 가장 첫번째는 해당 코드의 의존성을 제거할 수 있는지를 확인해보는 것이다. 의존성을 가지고 있다는 것은 후에 비즈니스 변경이 일어났을 때에도 좋지 않음으로 제거 할 수 있는 의존성이라면 처리하는 것이 좋다.

예를 들어 아래와 같이 덧셈을 위한 코드가 있다고 하자. 

```java

@Bean
public class Example {
    @Autowired
    Database database;
    @Autowired
    ExternalApi externalApi;

    public Integer add() {
        /* 데이터베이스에서 데이터를 가져온다 */
        Integer value1 = database.getValue();

        /* 외부 API에서 데이터를 가져온다 */
        Integer value2 = externalApi.getValue();

        return value1 + value2;
    }
}

```

이 코드에서 'value1' 값은 데이터베이스에서 가져오고 있고, 'value2' 값은 외부 API에서 가져오고 있다. 이 코드의 문제점은 이 두 값이 데이터베이스, 외부 API 에서 가져온다는 점이다. 즉 데이터베이스와, 외부 API에 의존성을 가지고 있다는 것이다. 이렇게 되면 이 두 영역은 우리가 관리할 수 있는 부분이 아니기 때문에 이 의존성을 제거할 수 없다.

테스트 코드는 항상 일관된 결과를 반환해야 한다. 하지만 위와 같은 의존성을 가진 코드가 테스트 커버리지 안에 존재하게 된다면 add() 함수는 의존성을 없애야함으로 테스트하기가 어려워진다. 아래는 예시의 테스트 코드이다.

```java

public class ExampleTest {
    @Autowired
    Example example;

    @Test
    public void add_test() {
        /* add() 함수의 결과가 데이터베이스, 외부 API 결과에 의존됨으로 일관된 결과를 가질 수 없다. */
        Integer result = example.add();
        assertThat(result).isEqualTo(10);
    }
}

```

이러한 문제를 개선하기 위해서는 비즈니스 코드와, 외부 코드를 구분할 필요성이 있다. 위에서 우리의 비즈니스 코드는 두 값을 더하는 부분이며, 외부 코드는 데이터베이스와, 외부 API를 호출하는 부분이다. 보통 이런 외부 코드는 함수의 parameter 로 받게 하여 비즈니스 코드와 구분을 짓고, 의존성을 관리할 수 있도록 한다. 아래는 의존성 문제를 개선한 add() 함수이다.

```java

@Bean
public class Example {
    @Autowired
    Database database;
    @Autowired
    ExternalApi externalApi;

    public Integer call() {
        /* 데이터베이스에서 데이터를 가져온다 */
        Integer value1 = database.getValue();

        /* 외부 API에서 데이터를 가져온다 */
        Integer value2 = externalApi.getValue();

        return add(value1, valued2);
    }

    public Integer add(Integer value1, Integer value2) {
        return value1 + value2;
    }
}

```

이렇게 작성하게되면 add() 함수의 관점에서는 외부에 종속된 코드는 모두 파라미터로 받았기 때문에 내부에서 항상 일관된 결과를 반환하는 순수함수로 작성되었다. 그렇기 때문에 테스트 코드도 작성이 가능하다.

```java

public class ExampleTest {
    @Autowired
    Example example;

    @Test
    public void add_test() {
        Integer value1 = 3;
        Integer value2 = 5;

        Integer result = example.add(value1, value2);
        assertThat(result).isEqualTo(8);
    }
}

```

이 테스트 코드의 목적은 우리의 비즈니스 코드인 add() 함수가 정상적으로 동작하는 것인지 확인하는 것이다. 여기에 외부에서(데이터베이스, API) 에서 값이 들어오는 과정은 중요하지 않다. 왜냐하면 이 값들은 외부에서 들어오기 때문에 우리가 테스트를 한다고 해서 우리가 수정할 수 있는 영역이 아니기 때문이다. 즉 이것 자체를 테스트하는 것은 무의미 하다.

### 의존성을 가진 코드를 테스트 하기

그럼에도 뭔가 꺼림직함을 느끼는 사람들이 있을 것이다. 왜냐하면 결국 외부 의존성을 가진 코드는 call() 함수가 가져갔고, 우리는 이 함수는 테스트하지 않았기 때문이다. 코드를 작성하다보면 불가피하게 의존성을 불러오는 코드쪽에도 비즈니스 코드가 들어가야할 때가 있다. 예를 들면 null 처리이다.

```java

@Bean
public class Example {
    @Autowired
    Database database;
    @Autowired
    ExternalApi externalApi;

    public Integer call() {
        /* 데이터베이스에서 데이터를 가져온다 */
        Integer value1 = database.getValue();

        if(Objects.isNull(value1)) {
            throw new Exception("'value1' is null");
        }

        /* 외부 API에서 데이터를 가져온다 */
        Integer value2 = externalApi.getValue();

        if(Objects.isNull(value2)) {
            throw new Exception("'value2' is null");
        }

        return add(value1, valued2);
    }

    public Integer add(Integer value1, Integer value2) {
        return value1 + value2;
    }
}

```

차선책은 있겠지만, 그래도 위와 같은 코드를 작성하려 한다면 비지니스 코드가 call() 함수에도 어느정도 내포되어 있기 때문에 테스트하고 싶을수도 있다. call() 함수에서 테스트하기 어렵게 만드는 것은 외부에 의존성을 가진 'database.getValue()', 'externalApi.getValue()' 이 두 함수인데, 이 둘을 대신할 가짜 객체를 생성해서 테스트할 수 있다. 가짜 객체를 만드는 첫번째 방법은 바로 추상체의 구현이다.

### 추상체의 구현

```java



```

### Mock 객체 활용

```java

public class ExampleTest {
    @Mock
    Database databaseMock;
    @Mock
    ExternalApi externalApiMock;
    @InjectMocks
    Example exampleMock;
    @Autowired
    Example example;

    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void call_test() {
        when(databaseMock.getValue()).thenReturn(3);
        when(externalApiMock.getValue()).thenReturn(5);

        Integer result = exampleMock.call();

        assertThat(result).isEqualTo(8);
    }

    @Test
    public void add_test() {
        Integer value1 = 3;
        Integer value2 = 5;

        Integer result = example.add(value1, value2);

        assertThat(result).isEqualTo(8);
    }
}

```

위와 같이 작성하면 메인 소스를 수정하지 않고 Mock 객체를 생성할 수 있으며 이를 활용해 의존성을 제거하고 비지니스 코드만을 테스트할 수 있다. @Mock, @InjectMocks 어노테이션들은 생성되어진 리얼 객체를 기반으로 가짜 Mock 객체를 덮어씌운다. 그렇기 때문에 database, externalApi 와 같이 Mock 주입을 할 객체들은 final 로 생성하면 Mock 주입을 받을 수 없다.

테스트를 위해서 메인 소스 코드가 변경되는 것은 옳지 않지만 아직 위의 케이스를 해결할 방법은 공부하지 못했다. PowerMock 이나 TestNG 라이브러리를 사용하면 된다고 하는데 다음에 찾아보도록 하자.



### 소스 분리
1. 외부 리소스를 활용하는 소스 분리

### 추상체 구현
2. 외부 리소스를 포함한 코드를 테스트하고 싶을떄에는 외부 리소스를 가져오는 객체의 추상체를
새롭게 구현하는 Stub 객체를 만들어서 테스트.
추상체가 없다면 이 방법은 사용 불가능.
추상체를 만들고, Stub 객체를 만들어야 하기때문에 1번방법보다는 보일러플레이트 코드가 많아짐
하지만 외부리소스를 포함한 코드도 테스트를 할수 있는 방법이 됨으로 테스트 커버리지가 높아짐.

내가볼때는 1, 2번 방법 모두를 사용하는 코드를 짜는게 좋은 코드일듯.

외부에서 주입을 할수 있도록 생성자 주입, Setter 주입 형태로 만들어 줘야함.

```java

	public PnrStatus.Res pnrStatus(PnrStatus.Req req) throws Exception {
		return this.pnrStatus(req, AirlineFactory.create(req.getAirline()));
	}

	private PnrStatus.Res pnrStatus(PnrStatus.Req req, Airline airline) throws Exception {
		if(req == null || StringUtils.isEmpty(req.getAirline())) {
			throw new Exception("파라미터 'req' 에서 항공사 코드를 가져올 수 없습니다");
		}

		return airline.pnrStatus(req);
	}

```

Stub 적용을 위한 외부소스 분리에는 리플렉션도 적용될 수 있을 것 같다.
위처럼 짜면 Airline 객체를 함수 외부 영역이기때문에 pnrStatus


### Mock, InjectMock 사용
파라미터도 같아야 합니다. -> 파라미터가 객체라면 이 객체도 Mock으로 잡아줘야한다. 객체끼리가 참조하는 곳이 다르면 다른 객체로 인식.
Mock끼리 의존성이 있는경우 의존된 객체의 함수들도 when().then() 으로 가짜 값을 반환하도록 만들어 줘야한다.
Mocking을 하려면 객체의 필드레벨로 해당 참조객체를 빼와야 하는데 Mocking할 대상이 만약 stateful한 객체라면 쓰레드 세이프하지 않을텐데.. 이럴뗀 어떻게??
이건 추후에 찾아봐야할듯.

컨트롤러 통합테스트할때는 서비스도 Mocking 해야함

final로 선언하면 Mock이 의존성 주입을 못하네.. 어떻게 해야하지.. -> 이거 꽤 심각한 이슈인데.. 우선은 컨트롤러쪽 테스트할때만 final 빼고 테스트하게끔 구성해야할듯

junit5, TestNG 등등 찾아봐야할듯 함

stub 이 필요한 객체를 테스트 커버리지의 영역 밖으로 뺄수 있으면 간단하게 테스트할수 있고 그 구조가 제일 좋은거같음
그런데 그렇지 못할 경우 @Mock, @InjectMock 을 활용해서 stub 함수를 주입해서 테스트.

MockMvcBuilders.standalone()... 요거 활용해서 컨트롤러에 목객체를 주입할 수 있다

한계점은 객체를 항상 클래스 레벨에서 노출을 시켜야 Mocking이 가능하다는 점. -> 리플렉션 되는 객체는 애초에 불가능

샘플

```java

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.mockito.Mockito.when;

    @Mock
    private AirfarePrice airfarePrice;
    @Mock
    private AirDemandTicket airDemandTicket;
    @InjectMocks
    private AirseoulTicketingService airseoulTicketingService;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }

        /** 'TKT634 OFFICE DATABASE MAXIMUM *DATE OVERFLOW*CONTACT SI' 오류 응답을 받을 때 적절히 파싱되는지 확인 */
    @Test
    public void ticketing_DbMaxoverflowErrorIsOccured() throws Exception {
        String airApiRequestInfoStub = "<AirApiRequestInfo Version=\"1.1\"><Config format=\"airseoul\" validationModule=\"VALIDATION_DOM_RS\" service=\"ticketing\"/><Response format=\"xml\" type=\"pc\"/><QueryString><![CDATA[authorizationCode=08086203&SEGIN=RS/905/202103261450/GMP/CJU/202103261600/Y/KRW/0/YHOW&idCode=ADT]]></QueryString><PnrService pnrAddr=\"HJ070\" pnrKey=\"500021127\"><AirlineCode>RS</AirlineCode></PnrService><PassengerService><PassengerList><PassengerData passengerType=\"ADT\" passengerKey=\"1\"><GivenName>기동</GivenName><Surname>윤</Surname><PTC>ADT</PTC></PassengerData></PassengerList></PassengerService><PaymentService><PaymentList><PaymentData type=\"CC\"><Amount>110000</Amount><CardCode><![CDATA[u3HUpnRURJ2uGkWnJ6lUkg==]]></CardCode><CardNumber><![CDATA[LjtbuPEUtStUn9nPD1ARfg==]]></CardNumber><Expiration><![CDATA[aXzFsA2YhFwQlQh/3LlCrA==]]></Expiration><Installment>00</Installment></PaymentData></PaymentList></PaymentService></AirApiRequestInfo>";
        AirApiRequestInfo param = (AirApiRequestInfo) jaxbMarshaller.unMarshall(AirApiRequestInfo.class, airApiRequestInfoStub);

        String airDemandTicketRSStub = "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"yes\"?><SOAP-ENV:Envelope xmlns:SOAP-ENV=\"http://schemas.xmlsoap.org/soap/envelope/\"><Header xmlns=\"http://schemas.xmlsoap.org/soap/envelope/\"><Conversation xmlns=\"http://sita.aero/hal\" id=\"conv-16p-1605634071703-147\" server=\"10.213.26.166\" timestamp=\"2020-11-17 17:27:51\"/></Header><SOAP-ENV:Body><SITA_AirDemandTicketRS xmlns=\"http://sita.aero/SITA_AirDemandTicketRS/4/0\" xmlns:common=\"http://sita.aero/common/2/0\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" Version=\"0\"><Errors><common:Error Code=\"101\" Tag=\"FETA\" Type=\"Host Error Response\">TKT634 OFFICE DATABASE MAXIMUM *DATE OVERFLOW*CONTACT SI</common:Error></Errors></SITA_AirDemandTicketRS></SOAP-ENV:Body></SOAP-ENV:Envelope>";
        airDemandTicketRSStub = airseoulCommonInfo.getBodyString(airDemandTicketRSStub)
                .replace(" xmlns=\"http://sita.aero/SITA_AirDemandTicketRS/4/0\"", "")
                .replace(" xmlns:common=\"http://sita.aero/common/2/0\"", "")
                .replace(" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"", "")
                .replaceAll("common:", "");
        SITA_AirDemandTicketRS airDemandTicketRS = (SITA_AirDemandTicketRS) jaxbMarshaller.unMarshall(SITA_AirDemandTicketRS.class, airDemandTicketRSStub);

        when(airfarePrice.getAirFarePriceRS(param)).thenReturn(new SITAAirfarePriceRS());
        SITAAirfarePriceRS airPriceRS = airfarePrice.getAirFarePriceRS(param);

        when(airDemandTicket.getAirDemandTicketRS(param, airPriceRS)).thenReturn(airDemandTicketRS);

        PaymentResult result = airseoulTicketingService.ticketing(param);

        log.info(jaxbMarshaller.marshall(result));
        assertThat(result.getErrorNo(), is("101"));
        assertThat(result.getErrorDesc(), is("TKT634 OFFICE DATABASE MAXIMUM *DATE OVERFLOW*CONTACT SI"));
    }

```

### Mock 에서 Real 객체 사용

```java

    @Mock(answer = Answers.CALLS_REAL_METHODS)
    MypageMapper mypageMapper;

```


### WireMock (Stub Server)
외부 서버도 일종의 가짜 서버를 두고싶을떄 사용할 수 있따.
위의 Stub을 만드는 방법은 결국 소스레벨에서 테스트하는 방법이기 떄문에. 스테이징 수준으로 테스트 레벨이 올라가게 되면
기획자나 다른 비개발자 직군은 테스트가 불가능하다. 
이럴떄는 가짜 데이터를 고정되게끔 줄수있는 스텁서버를 하나 구축을 해두면 테스트가 원활해질수 있다

### Static 클래스 Mocking


Stub 쓰는법 첫번째 -> 외부 자원들은 (리플렉션, API call, 특정 변수들, DB) 함수의 입력으로 가져오게끔 하라.
-> 의존성을 줄이는 방법중 첫번째이다.

Stub 쓰는법 두번째 -> 의존성을 제거할 수 없다면 Mocking을 하라