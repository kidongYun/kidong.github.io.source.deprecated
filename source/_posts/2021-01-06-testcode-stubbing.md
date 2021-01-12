---
layout: post
title:  "테스트 코드 작성시 stub 객체 생성에 대한 고찰"
date:   2021-01-06 20:00:00 +0900
categories: java
---

테스트 코드를 작성하다보면 서로 간에 의존성을 가지고 있어서 테스트를 하기 어려운 상황들을 직면하게 된다. 이를 해결했던 나의 경험들을 남겨두려고 한다. 이 글은 저자의 주관적인 생각이 포함되어 있음으로 맹목적인 신뢰는 지양하기 바란다.

### 외부 리소스 코드의 분리

테스트하려는 코드가 의존성을 가지고 있어서 테스트하기가 어렵다면, 이를 개선하기 위한 첫번째 방법은 해당 코드의 의존성을 제거할 수 있는지를 확인해보는 것이다. 의존성을 가지고 있다는 것은 후에 비즈니스 변경이 일어났을 때에 코드를 변경하기가 쉽지 않음으로 가능하다면 의존성은 제거하는 것이 좋다.

예를 들어 아래와 같이 덧셈을 위한 코드가 있다고 하자. 

```java

/* 데이터베이스에 접근하여 데이터를 가져오는 객체. */
class Database {
    public Integer getValue() {
        return new Random().nextInt();
    }
}

/* 외부 API 에 접근하여 데이터를 가져오는 객체. */
class ExternalApi {
    public Integer getValue() {
        return new Random().nextInt();
    }
}

class Example {
    Database database;
    ExternalApi externalApi;

    public Example(Database database, ExternalApi externalApi) {
        this.database = database;
        this.externalApi = externalApi;
    }

    public Integer add() {
        Integer value1 = database.getValue();
        Integer value2 = externalApi.getValue();

        return value1 + value2;
    }
}

```

이 코드에서 'value1' 값은 데이터베이스에서 가져오고 있고, 'value2' 값은 외부 API에서 가져오고 있다. 즉 데이터베이스와, 외부 API에 의존성을 가지고 있다는 것이다. 이렇게 되면 이 두 영역과 관련된 소스들 우리가 관리할 수 있는 부분이 아니기 때문에 이 의존성을 제거하는 것이 좋다.

테스트 코드는 항상 일관된 결과를 반환해야 한다. 하지만 위와 같이 의존성을 가진 코드가 테스트 커버리지 안에 존재하게 된다면 데이터베이스나, 외부 API의 변화에 따라서 코드는 동일하다 하더라도 일관된 결과를 얻지 못한다. 이렇듯 의존성을 가진 함수는 이 의존성 때문에 테스트하기가 어려워진다. 아래는 예시의 테스트 코드이다.

```java

public class ExampleTest {
    @Test
    public void add_test() {
        Example example = new Example(new Database(), new ExternalApi());

        /* add() 함수의 결과가 데이터베이스, 외부 API 결과에 의존됨으로 일관된 결과를 가질 수 없다. */
        Integer result = example.add();
        assertThat(result).isEqualTo(1);
    }
}
}

```

이러한 문제를 개선하기 위해서 생각해야할 것은 바로 비즈니스 코드와, 외부 코드를 구분할 필요성이 있다. 일반적으로 테스트를 해야할 코드는 우리의 비즈니스 코드이다. 즉 우리 관할안에 있는 코드를 테스트하고 관리해야한다. 위에서 우리의 비즈니스 코드는 두 값을 더하는 부분이며, 외부 코드는 데이터베이스와, 외부 API를 호출하는 부분이다. 보통 이런 외부 코드는 함수의 인수로 받게 하여 비즈니스 코드와 구분을 짓고, 의존성을 관리할 수 있도록 한다. 아래는 의존성 문제를 개선한 add() 함수이다.

```java

class Example {
    Database database;
    ExternalApi externalApi;

    public Example(Database database, ExternalApi externalApi) {
        this.database = database;
        this.externalApi = externalApi;
    }

    public Integer callAdd() {
        Integer value1 = database.getValue();
        Integer value2 = externalApi.getValue();

        return add(value1, value2);
    }

    public Integer add(Integer value1, Integer value2) {
        return value1 + value2;
    }
}

```

이렇게 작성하게되면 add() 함수의 관점에서는 외부에 종속된 코드는 모두 파라미터로 받았기 때문에 그 내부에서는 항상 일관된 결과를 반환할 수 있다. 그리고 일관된 결과를 얻을 수 있기 때문에 테스트 코드도 작성이 가능하다. 추가적으로 이런 일관된 결과를 반환하도록 함수를 순수함수라고 한다. (정확히는 순수함수 특성 중 하나) 순수함수의 형태로 함수를 작성하는 것은 의존성을 제거하는데에 좋다.

```java

public class ExampleTest {
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
 
class Example {
    Database database;
    ExternalApi externalApi;

    public Example(Database database, ExternalApi externalApi) {
        this.database = database;
        this.externalApi = externalApi;
    }

    public Integer callAdd() throws Exception {
        Integer value1 = database.getValue();

        if(Objects.isNull(value1)) {
            throw new Exception("'value1' must not be null");
        }

        Integer value2 = externalApi.getValue();

        if(Objects.isNull(value2)) {
            throw new Exception("'value2' must not be null");
        }

        return add(value1, value2);
    }

    public Integer add(Integer value1, Integer value2) {
        return value1 + value2;
    }
}

```

위와 같은 코드를 작성하려 한다면 비지니스 코드가 call() 함수에도 어느정도 내포되어 있기 때문에 테스트를 해야할 필요성이 생길수도 있다. 이렇게 call() 함수처럼 의존성을 가진 함수를 테스트하는 방법을 소개한다.

### 추상체의 구현

위 call() 함수에서 테스트하기 어렵게 만드는 것은 외부에 의존성을 가진 'database.getValue()', 'externalApi.getValue()' 이 두 함수인데, 이 둘을 대신할 가짜 객체를 생성해서 테스트할 수 있다. 가짜 객체를 만드는 첫번째 방법은 바로 추상체의 구현이다.

```java

interface Database {
    Integer getValue();
}

class DatabaseImpl implements Database {
    public Integer getValue() {
        return new Random().nextInt();
    }
}

interface ExternalApi {
    Integer getValue();
}

class ExternalApiImpl implements ExternalApi {
    public Integer getValue() {
        return new Random().nextInt();
    }
}

class Example {
    Database database;
    ExternalApi externalApi;

    public Example(Database database, ExternalApi externalApi) {
        this.database = database;
        this.externalApi = externalApi;
    }

    public Integer callAdd() throws Exception {
        Integer value1 = database.getValue();

        if(Objects.isNull(value1)) {
            throw new Exception("'value1' must not be null");
        }

        Integer value2 = externalApi.getValue();

        if(Objects.isNull(value2)) {
            throw new Exception("'value2' must not be null");
        }

        return add(value1, value2);
    }

    public Integer add(Integer value1, Integer value2) {
        return value1 + value2;
    }
}

class Main {
    public static void main(String[] args) throws Exception {
        /* 실제 소스에서는 추상체를 구현한 객체를 사용한다. */
        Example example = new Example(new DatabaseImpl(), new ExternalApiImpl());

        example.callAdd();
    }
}

class ExampleTest {
    @Test
    public void callAdd_test() throws Exception {
        /* 테스트코드에서는 추상체를 직접 구현하여 원하는 값을 반환하도록 함. */
        Database database = new Database() {
            @Override
            public Integer getValue() {
                return 3;
            }
        };

        /* 테스트코드에서는 추상체를 직접 구현하여 원하는 값을 반환하도록 함. */
        ExternalApi externalApi = new ExternalApi() {
            @Override
            public Integer getValue() {
                return 5;
            }
        };

        Example example = new Example(database, externalApi);
        Integer result = example.callAdd();
        assertThat(result).isEqualTo(8);
    }
}

```

추상체 구현방법을 활용해서 의존성을 가진 함수를 테스트하기 위해서는 우선 의존성을 내포한 객체 여기서는 Database, ExternalApi 를 추상화 시킨 객체가 필요하다. 그래서 두 타입의 interface 를 만들고, 이 인터페이스를 구현받아서 실제 소스에서 사용하는 코드를 작성한다. 그 부분이 Impl 이 붙은 부분이다. 테스트 시에는 이 인터페이스를 구현한 Impl 객체를 사용하지 말고 직접 구현하여 가짜 객체를 생성한다. 한 인터페이스로 묶여있기 때문에 이 가짜 객체가 callAdd() 함수에 넘겨졌을 때 정상적으로 동작할 것이다.

위 방식은 익명클래스를 활용한 방법인데, 가독성이 현저히 떨어진다. 만약 java8 버전 이상 사용하고 있다면 람다 표현식을 사용할 수 있으므로 이로 대체하자. 그렇다면 보다 훨씬 읽기 쉬워질 것이다.

```java

@FunctionalInterface
interface Database {
    Integer getValue();
}

@FunctionalInterface
interface ExternalApi {
    Integer getValue();
}

class ExampleTest {
    @Test
    public void callAdd_test() throws Exception {
        Database database = () -> 3;
        ExternalApi externalApi = () -> 5;

        Example example = new Example(database, externalApi);
        Integer result = example.callAdd();
        assertThat(result).isEqualTo(8);
    }
}

```

추상체 구현방법은 사실상 한계가 명확하다. 그 첫번째는 익명클래스를 사용하는 방법은 굉장히 가독성이 떨어졌다. 그래서 람다표현식을 활용했는데, 람다표현식은 한 인터페이스에 한 함수가 존재할때만 사용이 가능한 문법이기 때문에 만약 Database 객체와, ExternalApi 객체가 두개 이상의 함수를 가지고 있다면 람다표현식 사용이 불가능하다. 

두번째 문제가 더 중요한데 바로 테스트를 위해서 프로덕션 소스에 있는 객체의 추상체를 구현하면 안된다. 다시 말해서 단순히 테스트를 위해서 프로덕션 코드가 변경되면 안된다는 것이다. 이는 주객전도가 되는 일이며, 정말 그래야 하는 상황이 온다면 다시 생각해봐야한다. 

### Mock 객체 활용

이렇게 추상체를 구현해서 가짜 객체를 생성하는 방법은 제한사항이 많다. 이번에는 보다 널리 사용되는 Mock 라이브러리를 활용해보려고 한다. 이 라이브러리를 활용하면 추상체를 굳이 생성하지 않아도 가짜 객체를 생성하여 의존성을 대체할 수 있다. 세부적인 동작 원리는 잘 알지 못하지만, 기본적으로 Mock 라이브러리는 테스트시에 필요한 객체들이 생성되고 난 후 그 객체들에게 설정된 가짜 객체들을 대체 주입한다. 

```java

public class ExampleTest {
    @Mock
    Database databaseMock;
    @Mock
    ExternalApi externalApiMock;
    @InjectMocks
    Example exampleMock;

    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void callAdd_test() {
        when(databaseMock.getValue()).thenReturn(3);
        when(externalApiMock.getValue()).thenReturn(5);

        Integer result = exampleMock.callAdd();

        assertThat(result).isEqualTo(8);
    }
}

```

위와 같이 작성하면 메인 소스를 수정하지 않고 Mock 객체를 생성할 수 있으며 이를 활용해 의존성을 제거하고 비지니스 코드만을 테스트할 수 있다. @Mock, @InjectMocks 어노테이션들은 생성되어진 리얼 객체를 기반으로 가짜 Mock 객체를 덮어씌운다. 그렇기 때문에 database, externalApi 와 같이 Mock 주입을 할 객체들은 final 로 생성하면 Mock 주입을 받을 수 없다.

테스트를 위해서 메인 소스 코드가 변경되는 것은 옳지 않지만 아직 위의 케이스를 해결할 방법은 공부하지 못했다. PowerMock 이나 TestNG 라이브러리를 사용하면 된다고 하는데 다음에 찾아보도록 하자.

### ArgumentMatchers 라이브러리 활용

기본적으로 Mock 객체를 생성할 때 when() 절에 들어가는 파라미터도 모두 동일해야 실제로 가짜 객체 주입이 이루어진다.
그러나 이 파라미터들을 실제로 모두 맞추는 작업이 굉장히 번거롭다. 이런 상황을 위해서 mockito 라이브러리에서는 ArgumentsMatchers 객체를 제공하는데, 이 객체를 사용하면 파라미터의 값을 정확히 맞추지 않고 타입만 맞추면 가짜 객체 주입을 시킬 수 있다.

```java

/* ArgumentMatchers 사용하지 않는 경우. findById() 함수에 들어가는 파라미터도 동일하게 맞추어야 한다 */
when(objectiveService.findById(1L)).thenReturn(stub);

/* ArgumentMatchers 사용한 경우. Long 타입만 맞으면 자동 Mocking 된다 */
when(objectiveService.findById(anyLong())).thenReturn(stub);

```

### Mock 에서 Real 객체 사용

@Mock 어노테이션을 붙여서 생성된 객체들은 기본적으로 실제 로직을 사용하지 않는다. 만약 사용하고 싶다면 아래 옵션을 주면 된다.

```java

    @Mock(answer = Answers.CALLS_REAL_METHODS)
    MypageMapper mypageMapper;

```


### WireMock (Stub Server)

외부 서버도 일종의 가짜 서버를 두고싶을 때 사용하는 라이브러리이다. 위에서 언급한 Stub을 만드는 방법은 결국 소스코드 레벨에서 테스트하는 방법이기 떄문에. 스테이징 수준으로 테스트 레벨이 올라가게 되면 기획자나 다른 비개발자 직군은 테스트가 불가능하다. 이럴 때는 고정적인 가짜 데이터를 줄수 있는 Stub 서버를 하나 구축하는 방법도 있다.