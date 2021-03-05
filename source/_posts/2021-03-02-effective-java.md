---
layout: post
title:  "Interview Preparing"
date:   2021-03-02 00:0054 +0900
categories: java
---

### Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라.
장점
이름을 가질 수 있다.
호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
반환타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
valueOf : from과 of의 더 자세한 버전. 




### Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.
의존성을 이미 결정해버리기 때문에 다른 의존성을 주입할 수가 없다.
의존성 주입시 일반적으로 좋다라고 생각하는 방식은 생성자를 활용한 주입이다.
바로 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식.
의존객체를 주입하는 것은 유연성과 테스트 용이성을 높여준다.
이런 객체는 final을 선언하여 불변함을 보증해주면 다른 개발자들끼리 해당 객체를 안심하고 공유하고 사용할 수 있다..

의존 객체 주입 방식의 확장 - 팩토리 메서드 패턴을 전달하자.
Supplier<T> 객체활용해보기.




### Item 7. 다 쓴 객체 참조를 해제하라.
가비지 컬렉션이 메모리를 회수할 때 판단하는 첫번째 조건은 객체 참조 변수가 살아있는지이다.
그렇기 때문에 객체를 사용하고 난 후 참조를 해제하는 것이 중요하다.
계속 쌓이게 되면 성능에 악영향이 오고 나중엔 StackOverflow 오류가 발생한다.

선행적으로 Null을 검토하고 예외를 확인하는 습관은 중요하다.
물론 나중에 null로 인해 다른 곳에서 예외가 발생하니 이걸 굳이 먼저 체크할 필요가 있느냐라고도
생각할 수 있지만. 오류를 디버깅하는 관점에서 1번 라인에서null이발생하고 이로 인한 오류가 100번 라인에서
발발하게 되면 StackTrace 를 찍는다고 해도 1번 라인이 null이기 때문에 발생한 오류라고 직관적으로 대답해주지 못한다. 그렇기 떄문에 모든 오류는 가능한 조기에 발견하는 것이 디버깅하는데에 좋다.

또 이렇게 되면. Exception의 message를 커스텀하게 전달이 가능함으로 더 상세한 오류 내용을 알려줄 수 있다.

객체 참조를 해제하기 위해서 해당 객체에 null 을 밀어 넣는 것 보다는 해당 참조변수의 범위를 최소화 시키는 방법이 가장 좋다. 이 범위를 벗어나게 되면. 해제작업은 알아서 이루어진다.

일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.




### Item 10. equals는 일반 규약을 지켜 재정의하라.
아래케이스에 해당한다면 equals() 함수를 재정의할 필요가 없다.
1.각 인스턴스가 본질적으로 고유하다.
2. 인스턴스의 ‘논리적 동치성’을 검사할 일이 없다.
3. 상위 클래스에서 재정의한 equals가 하위클래스에도 딱 들어맞는다.
4. 클래스가 private이거나 package-private 이고 equals 메서드를 호출할 일이 없다.
5. 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스의 경우 equals 함수를 재정의하여 논리적 동치성을 보일 필요가 없다. 객체 식별자가 논리적 동치성이랑 동일하게 동작되기 때문.

equals() 함수를 재정의할 때 지켜야할 속성.
1. 반사성 : null이 아닌 모든 참조 값 x 에 대해, x.equals(x)는 true이다.
2. 대칭성 : null이 아닌 모든 참조 값 x,y 에 대해, x.equals(y)가 true이면 y.equals(x) 도 true 이다.
3. 추이성 : null이 아닌 모든 참조 값 x,y,z 에 대해, x.equals(y) 가 true, y.equals(z) true 이면 x.equals(z) 도 true이다.
4. 일관성 : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
5. null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다. 




### Item 11. equals를 재정의하려거든 hashCode도 재정의하라.

equal를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

2. equals(Object)가두 객체를 같다고 판단했다면, 두 객체의 hashCode는똑같은 값을 반환해야 한다.

3. equals(Object)가두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서롤 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블 성능이 좋아진다.

hashCode() 함수를 한가지 값만 반환하도록 구현하면 모든 객체에서 똑간은 해시코드를 반환하기 때문에 해시테이블의 버킷 하나에 모든 데이터들이 들어가기 때문에 연결 리스트 처럼 동작한다. 그결과 평균 수행시간이 O(1)인 해시테이블이 O(N)으로 느려진다.

해시의 속성을 잘 쓰기 위해서는 서로 다른 인스턴스에 다른 해시코드를 반환해야한다. 해시코드가 같다고 해서 두 객체가 같음을 의미하는 것은 아니다. 이것은 equals()함수로 처리하는 거다. hashCode() 함수는 말 그대로 해시테이블의 해시코드를 적용하는 것.

성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다. 속도야 빨라질 수 있겠지만 해시 품질이 나빠져서 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다. 핵심 품질을 떨어뜨린다는 의미는 해시테이블에 값들이 골고루 분포해야 해시 테이블의 성능을 활용 할 수 있는데 한곳에 몰리는 경우를 말한다.

hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다. 

AutoValue 프레임워크를 활용하면 equals(),hashCode() 함수들을자동으로 만들어준다.





### Item. 12 toString을 항상 재정의하라.






### Item 57. 지역변수의 범위를 최소화하라.
지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.

1. 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 ‘가장 처음 쓰일 때 선언하기’

또한 모든 지역변수는 선언과 동시에 초기화 해야한다. -> try-catch 구문을 사용할 때는 이 규칙을 지키기가 어렵다.
변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try블록안에서 초기화 해야하기 때문이고 또
변수 값을 try 블록 밖에서도 사용한다면. 이를 위해 지역변수의 범위를 try 블록 밖 까지 넓혀줘야하기 때문이다.

2. 반복문에서 범위 줄이기
while문 보다 for문이 가지는 장점 중 하나는 지역변수 반복자의 범위 제한이다.
while문을사용할 경우 반복자를 바깥쪽에서 초기화하고 반복문 안에서 사용하기 때문에 반복문 블록을 나와서도 반복자 변수를 사용할 수 있다.
for문은 이 반복자를 반복문 안에서 생성하기 떄문에 밖에서 사용할 수 없다. 보다 오류에 대해서 안전하다.
반복자의 범위를 제한하게 되었을 때 가지는 또하나의 장점은 반복자의 네이밍을 고민할 필요가 없다. 모두 동일한 이름을 가지더라도 모두 구분된다.
 
3. 메서드를 가장 작게 유지하고 한가지 기능에 집중하기.
한 메서드에서 여러가지 기능을 처리한다면 그 중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다. 해결책은 간단하다. 단순히 메서드를 기능별로 쪼개면 된다.




### Item 45. 스트림은 주의해서 사용하라.
스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.
람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
스트림으로 바꾸는게 가능할지라도 코드 가독성과 유지보수 측면에서 손해볼 수 있다.
스트림과 반복문을 적절히 조합하는게 최선
기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 떄만 반영.


스트림을 사용하기 어려운 케이스
파이프라인 작업 안에서 return, break, continue 문으로블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛰어야 할때.
스트림 파이프라인에서 바깥쪽의 지역변수를 사용하기 위해서는 final만 가능하다. 그렇기에 이를 수정해야하는 케이스라면 stream을 사용하기 어렵다.

좋은 케이스
원소들의 시퀀스를 일관되게 변환한다.
원소들의 시퀀스를 필터링한다.
원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
원소들의 시퀀스를 컬렉션에 모은다.
원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.
 
스트림을 반환하는 메서드 이름은 원소의 정체를 알려주는 복수 명사로 쓰기를 강력히 추천.





### Item 46. 스트림에서는 부작용 없는 함수를 사용하라.
스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 안흔다.
이렇게 하려면 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다.

스트림 종단연산 중 forEach() 는 스트림 계싼 결과를 보고할 때만 사용하고, 계산하는ㄷ ㅔ에는 사용하지 말자.


Item 58. 전통적인 for문보다는 for-each문을 사용하자.