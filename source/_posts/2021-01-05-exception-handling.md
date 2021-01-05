---
layout: post
title:  "예외 처리 방법에 대한 고찰"
date:   2021-01-05 20:00:00 +0900
categories: java
---

Java 프로그래밍 책을 맨 처음 펼치고 조금 읽다보면 'try-catch' 키워드를 활용한 예외처리 문법이 소개된다. 프로그래밍을 얼마 접하지 않은 초급 개발자들도 이 'try-catch' 키워드는 낯설지 않을 것이다. 그러나 정작 이 예외처리 문법을 보다 효율적으로 사용하는 할 수 있는 방법에 대해서는 쉽게 익혀지지 않는 것 같다. Java 프로그래밍 언어를 활용한 API 개발을 하면서 경험적으로 느낀 예외처리에 대한 나의 생각을 간단하게 적으려고 한다. 이 글은 저자의 주관적인 생각을 바탕으로 작성되었으므로 맹목적인 신뢰는 지양한다.

### 'try-catch' 키워드의 숨겨진 의미

```java

try {
    // 이 범위 안에서 무언가를 시도한다.
} catch(Exception e) {
    // 호출한 함수가 오류를 반환할 경우 예외처리 한다.
}

```

'무언가를 시도하고 뜻대로되지 않았을 때에는 오류를 잡아라.' 이런 느낌에서 'try-catch' 용어를 사용한 것 같다. 이 키워드는 결론적으로 이야기 하자면, 클라이언트 범주에 해당하는 코드에 사용된다. 클라이언트 코드라는 것은 어떤 특정 함수를 호출하는 함수이다. 그리고 이 반대의 개념으로 서비스 코드는 함수 호출 요청이 들어오면 특정 작업을 해주고 결과를 반환해준다. 이 개념은 우리가 흔히 아는 웹클라이언트, 웹서버의 개념과 동일하다. 사실 좀 더 폭넓게 생각해보면 우리가 작성하는 모든 코드는 이러한 클라이언트 코드와, 서비스 코드로 나눌수 있다.

```
클라이언트 코드 : 서비스 코드의 함수를 호출하는 함수
서비스 코드 : 클라이언트 코드의 요청을 받고 결과를 반환하는 함수
```

이 개념을 언급한 이유는 'try-catch' 이 키워드는 클라이언트 코드쪽에서만 사용되는 문법이라는 점이다. 계산기 객체를 활용해 간단한 예시를 들어보자.

```java

class Calculator {
    public add(int value1, int value1) {
        return value1 + value2;
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add(2, 5);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

위에서 Calculator 객체는 서비스 코드에 해당하고 Main 객체는 클라이언트 코드에 해당한다. Main 객체에서 Calculator 객체의 add() 함수를 호출하고 있으며, 이 함수가 어떤 결과를 반환할지 Main 객체의 입장에서는 알 수 없기 때문에 'try-catch' 키워드로 감싸고 있다. 위의 예제에서는 Calculator 객체의 add 함수가 오류를 반환한다고 보기 어렵지만 이해를 위해 add() 함수가 int 타입이 아닌 String 타입을 받게 만들어보자.

```java

class Calculator {
    public add(String value1, String value1) {
        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

이런 경우를 보면 왜 클라이언트 코드인 Main 객체가 'try-catch' 키워드로 감싸져야 하는 지가 보다 명확히 이해가 된다. 현재 우리 눈에는 Calculator 객체가 보이지만 실무에서 라이브러리를 활용하다보면 서비스 코드에 해당하는 객체를 쉽게 이해하기 어렵다. 위 add() 함수처럼 간단하지 않은 로직일 가능성이 높고, 또 라이브러리 안에 서비스 코드에 해당하는 함수를 찾는 것도 어렵기 때문에 알지 못하는 서비스 코드를 받는 클라이언트 코드는 'try-catch' 키워드를 사용해서 예외에 대비한다. 자 그러면 반대로 서비스 코드에서는 예외를 어떻게 다뤄야 할까?

위의 소스 코드를 다시 확인해보자. calculator.add() 두번째 호출은 숫자 타입이 아니기 때문에 오류가 반환됨을 알 수 있다. 그렇다면 이 오류는 누가 반환을 하는 것일까? 나도 정확하게는 파악하지 않았지만 Integer 객체의 parseInt() 함수 내부에서 오류를 반환하고 있을 것이다. 즉 Calculator 객체와, Integer 객체의 사이에서는 Calculator 객체가 클라이언트 코드가 되고, Integer 객체가 서비스 코드가 된다. 이렇듯 개발을 하면서 내가 짜는 코드가 클라이언트 코드가 될 수도 있고, 서비스 코드가 될 수도 있다. 
즉 add() 함수에서도 'try-catch' 키워드를 활용해 Integer 객체가 던지는 오류를 잡을 수도 있다.

```java

class Calculator {
    public int add(String value1, String value2) {
        try {
            return Integer.parseInt(value1) + Integer.parseInt(value2);
        } catch(Exception e) {
            return 0;
        }
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```

### 'throws' 키워드를 활용한 예외의 전파

위의 코드에서 add() 함수에서도 'try-catch' 키워드를 사용했고, main() 함수에서도 'try-catch' 키워드를 사용했다. 그러면 오류는 어떻게 될까. 오류는 add() 함수에서만 잡히고 main() 함수에서는 잡히지 않는다. 이유는 add() 함수가 프로그램 순서상 먼저 잡았기 때문이며, 이 예외라는 것이 'try-catch' 키워드로 잡히게 되면 전파되지 않는다. 그러면 만약 예외가 발생했을 경우 add() 함수에서 이 예외를 처리하지 않고 main() 함수에서 처리하고 싶다면 어떻게 해야할까? 이럴 때 'throws' 키워드를 사용한다.

```java

class Calculator {
    public int add(String value1, String value2) throws NumberFormatException {
        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

이렇게 작성하면 이전과 동일하게 main() 함수에서 예외가 잡힌다. throws 키워드가 특정 Exception에 한해서는 생략이 가능한데 NumberFormatException도 이에 포함되며, 그렇지 않는 Exception들이 더 많다.

### 'throw' 키워드를 활용한 예외의 시작

'try-catch' 키워드를 활용해 예외를 잡아냈고, 'throw' 키워드를 활용해 예외를 전파했다. 그러나 이쯤되면 생각이 드는 것이 그러면 예외는 과연 어디서부터 시작하는 것일까? 라는 것이다. 예외의 시작은 'throw' 키워드를 통해서 만들어낼 수 있다. 이 키워드는 'try-catch' 키워드와는 반대로 서비스 코드에서만 사용이 된다. 'throw', 'throws', 'try-catch' 이 3가지의 키워드가 서로 조화를 이뤄서 Java 에서는 예외처리를 할 수 있다.

```
'throw' : 예외의 출발 지점
'throws' : 예외의 중간 지점
'try-catch' : 예외의 도착 지점
```

예외를 만들어내는 간단한 예제를 생각해보자.

```java

class Calculator {
    public int add(String value1, String value2) throws Exception {
        if(value1.replaceAll("[^0-9]", "").isEmpty()) {
            throw new Exception();
        }

        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

add() 함수에서 value1 값에 숫자가 아닌 값들은 지우고, 지우고 난 뒤에 숫자가 없다면 예외를 발생시켜 전파하는 예제이다. 이 전파된 예외는 main() 함수에서 잡아서 처리된다. 예외처리는 이렇게 이 3가지 키워드가 전부이며 기본적으로 이 규칙만 이해하면 된다.

그렇다면 언제 예외를 생성해야 할까? 클라이언크 코드 관점에서는 서비스 코드가 어떤 값을 반환할지 모르기 때문에 불안정 요소라면 반대로 서비스 코드 관점에서는 클라이언트 코드의 입력이 어떻게 들어올지 모르기 때문에 이 클라이언트 코드의 입력이 불안정 요소이다.

```java

/* parameter 라는 값은 클라이언트 코드에서 받아오는 값이며, example() 이라는 서비스 코드에게는 알 수 없는 미지의 값이다.*/
String example(String parameter) {

    /* service.call() 함수는 반대로 서비스 코드에게서 반환되는 값이며, 이 또한 example() 이라는 클라이언트 코드에게는 알 수 없는 미지의 값이다. */
    return service.call(parameter);
}

```

위와 같이 불안정 요소에 대해서 조치를 해야할 때 예외 처리를 해야할 필요성이 생기며 처리 가능 여부에 따라서 조치 내용이 다르다.

```

처리가 가능한 예외가 발생한 경우
-> 보완 로직을 수행하도록 분기한다.

처리가 불가능한 예외가 발생한 경우
-> 상세 에러 메시지를 포함하여 예외를 던진다.

```

### 클라이언트 코드에서 예외처리를 하는 것이 보다 유연하다

어쩌다보니 예외처리에 대한 기본적인 문법도 위에서 언급하게 되었는데, 예외처리를 위한 각 키워드들이 가지고 있는 목적을 좀더 명확하게 하는데 도움이 되는거 같아 함께 적었다. 내가 쓰고싶었던 글은 지금부터의 글인데, 저자가 스프링 프레임워크 기반으로 API 개발을 할 때 예외처리를 어떻게 하는 것이 보다 유연하고 효율적일까를 고민한 것으로 API 개발에 종속적인 내용일 수 있다. 보통 스프링 프레임워크에서 클라이언트 코드라고 하면 Controller 영역을 이야기하고, 서비스 코드라고 하면 Service 영역을 이야기하는데,  예외처리는 Service 영역에서 하는 것 보다 Controller 영역에서 하는 것이 보다 더 유연하다.

한가지 예제 소스와 함께 이해해보자. 위와 동일하게 Calculator 객체가 있고, 이 객체를 User1, User2 객체가 사용하고 있다. 즉 User1, User2 두 객체가 클라이언트 코드가 되고, Calculator 객체가 서비스 코드가 된다. 현재 예제에는 Calculator 객체에서 'try-catch' 키워드를 활용해 예외 처리를 끝냈으며, 이 예외는 User1, User2 두 객체에게 전달되지 않는다. 여기서 발생하는 문제점은 User1, User2 두 객체가 add() 함수에서 오류가 발생했을 때 그에 대한 조치를 각자 비지니스에 맞게 조절할 수 없다는 것이다.
만약 User1 객체에서는 calculate() 함수를 호출하고 오류가 났다면 기본 값으로 0을 주고 싶고, User2 객체에서는 기본 값으로 -1을 주고 싶어 한다면 User2 객체는 예외가 발생했는지 안했는지를 모르기 때문에 이 -1 값을 줄 수 없다. 

```java

class Calculator {
    public int add(String value1, String value2) throws Exception {
        try {
            return Integer.parseInt(value1) + Integer.parseInt(value2);
        } catch (Exception e) {
            return 0;
        }
    }
}

class User1 {
    Calculator calculator = new Calculator();

    public void calculate() {
        calculator.add("2", "5");
    }
}

class User2 {
    Calculator calculator = new Calculator();

    public void calculate() {
        calculator.add("2", "5");
    }
}

```

이처럼 예외에 대해서 각 클라이언트 코드가 어떻게 조치할지는 비지니스에 따라서 달라질 수 있기 때문에 예외처리는 최대한 클라이언트쪽에서 하는 것이 보다 유연해진다. 그래서 Spring Framework 를 사용하는 경우 대부분 서비스, 데이터 액세스 레벨, 외부 데이터 연동 등과 같은 구간에서는 예외를 'throws' 키워드를 활용해 전파하기만 하고 예외를 처리하지는 않는다. 예외를 실제 처리하는 부분은 가장 클라이언트 코드인 Controller 영역에서 예외처리를 담당한다. 그렇게 하는 것이 보다 유연하기 때문이다.

```java

public class Controller {
    @GetMapping("/api1")
    public ResponseEntity<?> api1() {
        try {
            return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        }
    }

    @GetMapping("/api2")
    public ResponseEntity<?> api2() {
        try {
            return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        }
    }
}

```

### @Advice

그러면 Contoller 보다 더 클라이언트에 가까운 코드는 없을까? 사실은 없다. 왜냐면 비지니스 관점에서 Controller 다음 영역은 프론트 엔드 영역으로 넘어가기 때문이다. 하지만 Spring Framework 에서는 @Advice 어노테이션을 활용해 Controller 보다 더 클라이언트 코드인 영역을 제공한다. 사실 @Advice 의 존재는 위에서 내가 설명한 것과 조금은 역설적이다. 왜냐하면 예외처리를 클라이언트 코드 쪽에서 해야한다고 한 이유는 보다 더 비지니스에 유연할 수 있기 때문이라고 했는데, @Advice 의 존재는 비지니스와는 관련이 없기 때문이다. 

클라이언트 코드로 예외처리를 넘기게되면 생기는 단점은 유연성이 많이 생기는 만큼 그만큼 코드의 중복이 생긴다는 점이다. 위에서 언급한 Controller 코드 예시도 예외처리 부분에서 결국 동일한 대처를 하고있기 때문에 코드가 중복된다고 볼 수 있다. 즉 유연성이 늘어난 만큼 이를 활용하지 않고 모두 동일한 예외처리를 한다면 이는 모두 중복된 코드가 될 수 있다는 것이다. 이러한 중복을 개선시키기 위해서 @Advice 어노테이션이 등장했다.

```java

// Advice
@RestControllerAdvice
public class Advice {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> exception(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
    }
}


// Controller
public class Controller {
    @GetMapping("/api1")
    public ResponseEntity<?> api1() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }

    @GetMapping("/api2")
    public ResponseEntity<?> api2() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }
}

```

Controller 에서도 'try-catch' 키워드를 활용해 예외처리를 하지 않는다면 이 예외는 전파되어 Advice 객체에서 처리된다. @ExceptionHandler 어노테이션에 지정된 예외클래스로 들어온 경우에 매핑되어 구현된 동작이 실행된다.

### Exception.class 의 한계

기본적으로 모든 예외 클래스들은 Exception.class 를 상속받는다. 그래서 'try-catch' 로 예외처리를 할 때에도 catch 영역에 Exception.class 를 주게되면 모든 예외 클래스들이 들어오게 됨으로 간편했다. 자 그러면 이렇게 널리 공용으로 사용되는 Exception.class 만으로는 해결할 수 없는 문제를 생각해보자.

공통으로 사용하는 예외 클래스들 같은 경우 범용적으로 사용해야 하기 때문에 오류를 반환하는 내용들이 굉장히 추상적이다. 그래서 내가 구현하려고 하는 API 서버에 예외가 발생했을 때 발생한 예외에 대해서 구체적인 로그를 남겨두려고 한다. 아래 예제 코드를 참고하여 이해해보자.

```java

class Calculator {
    public int add(String value1, String value2) throws Exception {
        if(value1.replaceAll("[^0-9]", "").isEmpty()) {
            throw new Exception("value1 변수에 적절하지 않은 값이 입력되었습니다.");
        }

        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

위와 같이 작성하면 Exception 객체 생성시 같이 전달된 Message 정보가 stackTrace 에 남게 된다. StackTrace 를 남겨보면 알겠지만 함수의 콜백이 모두 나오기 때문에 내용이 많아 실무에서 이를 로그로 사용하기는 쉽지 않다. 그래서 이번에는 stackTrace 를 찍지 않고 단순 로그를 남겼다.

```java

class Calculator {
    public int add(String value1, String value2) throws Exception {
        if(value1.replaceAll("[^0-9]", "").isEmpty()) {
            throw new Exception("value1 변수에 적절하지 않은 값이 입력되었습니다.");
        }

        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자") // 오류 반환
        } catch (Exception e) {
            log.info(e.getMessage());
        }
    }
}

```

여기서 문제점은 발생한다. 만약 우리가 'throw' 키워드를 활용해서 던진 예외의 경우 어떤 케이스 인지하고 서비스 코드에서 예외를 직접 던진 것이기 때문에 로그만 남겨도 충분하지만 만약 그 외에 예상하지 못한 예외들이 던져졌을 경우 이 로그만을 보고 원인 파악을 하는 것은 쉽지 않다. 즉 내가 언급하고 싶은 것은 내가 알고 직접 던진 예외와, 나도 모르게 던져진 예외는 처리가 달라야 함을 인지해야 한다는 것이다. 그러나 Exception.class 와 같이 공통적으로 사용하는 예외 클래스들을 사용하면 두 가지의 방법의 겹쳐지기 때문에 둘을 구분할 수 없다.

### 커스텀한 예외 클래스를 만들자.

이 문제를 해결하기 위해서 새로운 예외 클래스를 만들자. 이 예외 클래스는 나의 API 서버에서만 사용되는 예외 클래스이기 때문에 다른 클래스와 겹쳐질 걱정을 안해도 된다.

```java

class CustomException extends Exception {
    private final String message;

    public CustomException(String message) {
        this.message = message;
    }

    public String getMessage() {
        return this.message;
    }
}

class Calculator {
    public int add(String value1, String value2) throws Exception {
        if(value1.replaceAll("[^0-9]", "").isEmpty()) {
            throw new CustomException("value1 변수에 적절하지 않은 값이 입력되었습니다.");
        }

        return Integer.parseInt(value1) + Integer.parseInt(value2);
    }
}

class User {
    Calculator calculator = new Calculator();

    public void calculate() {
        try {
            calculator.add("2", "5");   // 정상 케이스
            calculator.add("2", "글자"); // 오류 반환
        } catch(CustomException e) {
            System.out.println(e.getMessage());
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}

```

내가 인지하지 못하는 예외 클래스들은 Exception.class 를 받는 catch 영역에서 stackTrace() 를 호출하고 있고, 내가 직접 던진 예외의 경우 CustomException.class 를 받는 catch 영역에서 로그만 남기도록 되어 있다. 이렇게 내가 직접 던진 예외와, 내가 모르는 예외를 구분하여 작업할 수 있다. 잘 생각해보면 우리가 쓰는 라이브러리들도 각각 개인적으로 정의하고 있는 예외 클래스들이 있다. 이처럼 각자의 프로젝트에 맞게 예외 내용들을 구체화 시켜야 한다면 이렇게 커스텀한 예외 클래스를 만들어서 사용하여야 한다.

### API 서버에서 사용하는 커스텀 예외 클래스

이제는 API 서버에 종속적으로 예외 클래스를 활용하는 방법을 언급하려고 한다. 전통적인 API 서버는 API의 응답 코드를 프로젝트마다 정의하고 있다. 예를 들면 에러코드 '1000' 이면 '연동 오류' 이런식으로 말이다. API 서버에 맞는 커스텀 예외 클래스를 사용하려면 우선적으로 있어야 하는 것은 프로젝트 전반에 걸쳐서 공통적으로 사용하는 응답코드 이다. 그리고 이 응답코드를 포함해 커스텀 예외 클래스를 만들어 낸다.

```java

enum ApiCode {
    SUCCESS(0, "성공"), PAYMENT_FAIL(1000, "결제 오류"), RESERVATION_FAIL(2000, "예약 오류");

    ApiCode(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    private final int code;
    private final String desc;

    public int code() { return code; }
    public String desc() { return desc; }
}

class ApiException extends Exception {
    private final ApiCode apiCode;

    public ApiException(ApiCode apiCode) {
        this.apiCode = apiCode;
    }

    public ApiCode apiCode() {
        return this.apiCode;
    }
}

public class Controller {
    @GetMapping("/api1")
    public ResponseEntity<?> api1() {
        try {
            return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
        } catch (ApiException e) {
            return ResponseEntity.status(e.apiCode().code()).body(e.apiCode().desc());
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        }
    }

    @GetMapping("/api2")
    public ResponseEntity<?> api2() {
        try {
            return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
        } catch (ApiException e) {
            return ResponseEntity.status(e.apiCode().code()).body(e.apiCode().desc());
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
        }
    }
}

```

이러한 커스텀 예외 클래스도 @Advice 객체에 전파시킬 수 있다.

```java

// Advice
@RestControllerAdvice
public class Advice {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> exception(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
    }

    @ExceptionHandler(ApiException.class)
    public ResponseEntity<?> apiException(ApiException e) {
        return ResponseEntity.status(e.apiCode().code()).body(e.apiCode().desc());
    }
}


// Controller
public class Controller {
    @GetMapping("/api1")
    public ResponseEntity<?> api1() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }

    @GetMapping("/api2")
    public ResponseEntity<?> api2() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }
}

```

### REST API 서버에서 사용하는 커스텀 예외 클래스

드디어 이 글의 마지막 단계에 도달했다. 전통적인 API 서버들은 위에서 언급한 것처럼 각 프로젝트마다 errorCode, errorDesc 를 개별적으로 정의하고 사용한다. 그러나 현대에는 HttpStatus 코드를 활용하여 이를 대체하는데, 모두가 아는 코드이기 때문에 새로운 학습비용이 들지 않는다는 장점이 있다. 반대로 온전히 비지니스에 맞는 에러코드를 정의할 수 없다는 점이 단점이다. 그렇기 때문에 API 프로젝트 성향에 따라서 위와 같은 커스텀 예외 클래스를 작성해도 되며, HttpStatus Code 규칙을 따르겠다고 한다면 아래와 같은 예제로 작성할 수있다. 이미 응답코드가 정의되어 있기 때문에 커스텀 예외 클래스가 이미 이 응답코드에 맞게 정의되어 있다. 이를 활용하면 훨씬 간편하다.

```java

// Advice
@RestControllerAdvice
public class Advice {
    /** Common Exception Handler */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> exception(Exception e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());
    }

    /** Custom Exception Handler */
    @ExceptionHandler(HttpStatusCodeException.class)
    public ResponseEntity<?> httpStatusCodeException(HttpStatusCodeException e) {
        return ResponseEntity.status(e.getStatusCode()).body(e.getStatusCode().getReasonPhrase());
    }
}

// Controller
public class Controller {
    @GetMapping("/api1")
    public ResponseEntity<?> api1() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }

    @GetMapping("/api2")
    public ResponseEntity<?> api2() {
        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase);
    }
}

```