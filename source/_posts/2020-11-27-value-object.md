---
layout: post
title:  "Enum"
date:   2020-11-27 11:38:54 +0900
categories: java
---

Java 에서 제공하는 Enum 타입은 Value Object 를 표현하는데 굉장히 유용하다. 이 글에서는 Enum 을 활용해서  Value Object 를 가장 잘 표현하는 방법을 향해 가는 과정을 남기려고 한다. 이 글은 필자의 주관적인 관점을 바탕으로 작성된 글임으로, 단순히 하나의 참고자료로 봐주었으면 좋겠다.

### Value Object 란?

우선 Value Object 에 대해서 먼저 간략히 이해해 보자. 일반적으로 주로 자바 프로그래밍 언어에서 언급되는 개념이며 '행위가 없는 객체'를 의미한다. 또 이 단어는 'VO' 라는 약자로 주로 사용된다. 행위가 없다는 말은 오로지 상태만을 가지고 있다는 의미이며, 이 객체들이 가진 상태는 일반적으로 유한하고, 제한된 상태 값만을 가진다. 시스템의 조건에 따라 다를수는 있지만 대표적으로 동전, 돌멩이, 계절 등이 있을 수 있다. 이와 같이 대부분 피사체를 표현할 때 VO 로써 바라보게 된다.

```
Value Object = 

    (1) 행위가 없는 객체
    (2) 특정 상태 값을 가지는 객체

```

### Enum 없이 Value Object 를 표현해보자.

기본적으로 자바 언어를 배우게 되면 원시타입이 아닌 사용자 레벨에서 필요한 객체를 만들기 위해 맨 처음으로 'Class' 타입을 배운다. 이 키워드는 Java 에서 사용하는 모든 커스텀 타입의 기본이 됨으로 이를 사용해서도 Value Object 를 표현할 수 있다. 아래는 'Class' 를 활용하여 완벽하진 않지만 가장 간단하게 사계절을 표현하는 객체를 만든 것이다.

```java

class Season {
    String SPRING = "SPRING";
    String SUMMER = "SUMMER";
    String FALL = "FALL";
    String WINTER = "WINTER";
}

class Main {
    public static void main(String[] args) {
        String season = Season.SPRING;
    }
}

```

Season 이라는 클래스를 만들고 여기에 사계절의 이름을 담은 String 타입의 필드를 생성했다. 이러한 형태로 Value Object를 표현하는 것이 과연 적절했는지 Value Object의 특성을 비교해보며 확인해보자. 우선 메소드를 정의하지 않고 필드만 선언하였음으로 행위가 없는 객체임은 맞지만, String 타입은 어떠한 값이든 받을 수 있기 때문에 특정한 상태 값만 가질 수 있다는 Value Object 의 두번째 특성을 만족시키지 못했다. 아래는 이러한 문제를 개선시킨 코드이다.

```java

class Season {
    public static final Season SPRING = new Season();
    public static final Season SUMMER = new Season();
    public static final Season FALL = new Season();
    public static final Season WINTER = new Season();
}

class Main {
    public static void main(String[] args) {
        Season season = Season.SPRING;
    }
}

```

### Enum 으로 Value Object 를 표현해보자.

위 코드 방식은 Value Object 의 두 특성 조건을 만족한다. Java 언어에서는 위와같은 상수의 열거형을 표현하기 위해서 enum 이라는 문법을 제공한다.
이 문법을 사용하면 보다 심플한 방법으로 위와 동일한 소스를 구현할 수 있다.

```java

enum Season {
    SPRING, SUMMER, FALL, WINTER
}

class Main {
    public static void main(String[] args) {

    }
}

```


이러한 이슈를 해결하기 위해서 타입스크립트에서는 위와 같은 문법을 제공한다. 봄, 여름, 가을, 겨울 이라는 특수한 문자열 값을 타입으로써 사용할 수 있다. 그리고 이들을 OR 연산자로 묶어 이 네 개의 문자열만 들어올 수 있도록 표현한다. 이러한 표현 방법은 명확히 눈에 들어온다. 그러나 자바에서는 타입스크립트과 같이 값을 타입으로써 사용하는 문법이 존재하지 않는다. 그러면 Value Object 타입을 어떻게 만들 수 있을까? enum 문법을 사용하여 구현이 가능하다.

```java

class Season {
    enum name {
        SPRING,
        SUMMER,
        FALL,
        WINTER
    }
}

class Main {
    public static void main(String[] args) {
        Season season = new Season();
        season.name = Season.name.WINTER;
    }
}

```

위에서 만든 객체에서 enum 타입의 name 필드는 SPRING, SUMMER, FALL, WINTER 4가지 값만 가질 수 있다. 이는 typescript 에서 보여준 것과 동일한 기능을 한다.

### 한글명도 가진 사계절 Value Object

```java

```



```java

class Something {
	String season;
}

class Main {
	public static void main(String[] args) {
		Something something = new Something();
		
		something.season = “봄”
		something.season = “여름”
		something.season = “가을”
		something.season = “겨울”

		something.season = “계절이 아닙니다”
	}
}
```

위 방식이 타입스크립트에서 처음으로 제시한 방법을 자바로 작성한 소스코드이다. 타입스크립트와 동일하게 String 타입으로 구현되어 있어서 어떤 값이든 다 들어갈수 있는 것이 문제이다. 사실 커스텀 타입을 사용하지 않고도 원하지 않는 값이 특정 필드에 값이 들어가지 않게 끔 하기 위해서 보통 자바빈즈 패턴(getter/setter) 을 사용한다. 필드에 직접 접근하게 하는 것이 아니고 메서드를 통해 접근하도록 해서 들어오는 값에 대한 검증 처리를 하는 것이다.

```java

class Something {
	private String season;
	
	public void setSeason(String season) {
		if(!“봄여름가을겨울”.contains(season)) {
			return;
		}

		this.season = season;
	}
}

class Main {
	public static void main(Stirng[] args) {
		Something something = new Something();

		something.season = “봄”;
	}
}

```

# 주제. enum의 형태

1. 단일 타입.

2. 맵 타입
```java

    public enum Type {
        SCHEDULE("S"), MYPAGE("M");
        Type(String value) { this.value = value; }

        private final String value;
        public String value() { return value; }
    }

```

3. enum 타입 jackson serial, unserial 하기.

```java

    public enum Type {
        SCHEDULE("S"), MYPAGE("M");
        Type(String value) { this.value = value; }

        private final String value;

        @JsonValue
        public String value() { return value; }
    }
```

4. 속성을 계속 늘릴수 있다.

```java

    public enum Type {
        SCHEDULE("S", "스케줄"), MYPAGE("M", "마이페이지");
        Type(String value, String desc) {
            this.value = value;
            this.desc = desc;
        }

        private final String value;
        private final String desc;

        @JsonValue
        public String value() { return value; }
        public String desc() { return desc; }
    }

```

@JsonValue annotataion을 붙인다.


# 주제 Component 와 Data
Component 설계할때 데이터에 영향이 있는것은 주입받도록 만들고 나머지는 해당 컨테이너에서 처리하면된다.
추상화 기법은 Component 작업할때 별로 좋지 않은것 같다.
styled 와 관련된 Component 를 두고 예를 추상화 시켜서 쓰려고 했는데. 결국 css 문법을 계속 한번씩 확인하게 되고.

width, height 등과 관련된 Layout Component 하나 두고 이걸로 저런 값들을 조절하고

button, label 등의 글자나 특색이있는 UI 요소들은 다른 component로 둬서 둘을 결합해서 사용하는게 좋은듯.

이런식으로 기본적으로 내가 만든 모든 컴포넌트는 레이아웃에 특별함이 필요하다하면 저걸로 감싸는거지.
<LayoutComponent>
    <ButtonComponent>
</LayoutComponent>

레이아웃적인 요소와 특정 컴포넌트만 가지는 요소를 구분시키고 이 레이아웃적인 요소를 감싸게끔 만드는게 진짜 조은듯