---
layout: post
title:  "Present Value Object using Enum"
date:   2020-11-27 11:38:54 +0900
categories: java
---

### Value Object 란?

일반적으로 Value Object는 주로 자바 프로그래밍 언어에서 언급되는 개념이며, '변하지 않는 객체'를 의미한다. 물론 현실적으로 변하지 않는 객체를 만든다는 것은 비약일 수 있지만 여기서 우리가 집중해야할 것은 실세계에서가 아닌 어떠한 시스템 관점에서 객체를 표현할 때 그것이 변하는가 안변하는가를 의미한다. 일반적인 쓰여지는 예시로는 동전, 돌멩이, 계절 등이 있을 수 있다. 즉 다시 말하면 쉽게 변하지 않는 객체인 것이다.

### 타입스크립트로 계절를 표현해보자

```typescript

class Season {
    name: string
}

/** 사계절 정보를 넣은 경우 */
new Season("WINTER");

/** 사계절 정보를 넣지 않은 경우 - 의도하지 않았지만 데이터가 들어간다. */
new Season("계절이아닙니다");

```

위 코드는 타입스크립트 언어를 활용해 사계절 객체를 표현한 것이다. 이 객체는 각 계절의 이름을 표현하는 오직 'name' 필드만 string 타입으로 가지고 있다.
일반적으로 사계절은 '봄', '여름', '가을', '겨울' 총 4가지의 name만을 생각할 수 있고 이는 쉽게 변하지 않는다. 즉 Value Object의 개념으로 볼 수 있다. 지금 코드의 문제점은 이 name 필드의 타입이 string 이라는 점이다. 이는 사계절에 해당하는 정보가 아닌 다른 형태의 무수한 값들도 아무런 제약 없이 입력이 가능하다. 즉 사계절 값이 들어갈 것이라는 정책적인 약속만 존재할 뿐 이를 소스 코드에서 제한하고 있는 부분은 없다. 정책을 모르고 이 소스를 처음 사용하는 개발자는 사계절에 해당하지 않는 다른 정보도 넣을 수 있는 여지가 생길 것이다.

### 봄, 여름, 가을, 겨울 만 입력될 수 있도록 제한해보자

```typescript

class Season {
    name: 'SPRING' | 'SUMMER' | 'FALL' | 'WINTER'
}

/** 사계절 정보를 넣은 경우 */
new Season("WINTER");

/** 사계절 정보를 넣지 않은 경우 - 오류 발생 */
new Season("계절이아닙니다");

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