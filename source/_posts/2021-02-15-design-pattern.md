---
layout: post
title:  "Design Pattern"
date:   2021-02-15 00:0054 +0900
categories: java
---

### 디자인패턴 vs 알고리즘

알고리즘은 문제 해결을 위한 처리 절차.

ex 1) 서울에서 부산으로 가는 방법
1. 걸어서 간다
2. 자전거를 타고 간다.
3. 버스를 타고 간다.
4. KTX를 타고 간다.
5. 비행기를 타고 간다.

ex 2) 여러 개의 데이터가 있을 때 순서에 맞게 정렬하는 방법
1. 단순 정렬
2. 버블 정렬
3. 삽입 정렬
4. 쉘 정렬
5. 퀵 정렬

디자인 패턴

효율적인 프로그래밍을 하고 있다는 판단하는 기준
1. 중복된 코드가 없다.
2. 변경된 요구사항을 순조롭게 받아들일 수 있어야 한다.
(이 둘중에 어떤게 더 우선이 될까??)

디자인 패턴을 적용하여 프로그램을 만드는 중요한 이유는 다양한 추가 요구사항에 대해 좀 더 쉽게 대응하기 위해서이다.
= 과거에 이런 상황에 이렇게 하니 요구사항 변경에 따라 코드 수정을 하기 좋았다.

디자인패턴도 같은 상황에 하나의 패턴만이 정답은 아니다.

디자인패턴도 만드는 서비스의 종류에 따라서 다양하게 나뉜다.
1. 오리지널 파운데이션 패턴(GoF 패턴)
2. 썬의 J2EE 패턴
3. JSP 패턴
4. 아키텍쳐 패턴
5. 게임 디자인 패턴

GoF 패턴에는 아래와 같은 패턴들이 있음.
1. 싱글턴 패턴
2. 플라이웨이트 패턴
3. 빌더 패턴
4. 옵저버 패턴
5. 어댑터 패턴
6. 파사드 패턴
7. 데코레이션 패턴
8. 브릿지 패턴
9. 스트래티지 패턴


### 디자인 패턴과 클래스의 다형성

1. 참조타입 사이에서의 형변환
클래스 사이에서도 형변환이 일어나는데, 클래스 사이의 형변환은 상위클래스와 하위클래스 사이에서 일어나게 된다.
-> 하위클래스의 객체는 상위클래스의 객체에 대입할 수 있다.
-> 상위클래스 객체를 하위클래스 객체에 대입할 수 없다.
: 다형성의 특징.

객체지향 프로그래밍을 하는데 굉장히 큰 근간이 되는 특징이 다형성이다.

### 클래스의 약한 결합, 강한 결합의 이해

DI(Dependency Injection) 의존성 주입 : 객체지향 프로그래밍에서 다른 객체를 사용하는 것을 다른 객체에 의존한다고 표현.
다른 객체를 사용하기 위해서는 다른 객체를 생성해야 하는데. 직접 생성도 가능하고, 미리 생성되어있는 다른 객체를 받아올수도 있다.
이런 것들이 모두 의존성 주입의 방법들

```java

class A {
    B b;
    C c;

    public A(C c) {
        this.c = c;
    }

    // Setter 주입
    public setB(B b) {
        this.b = b;
    }
}

```

생성자 주입이나, Setter 주입은 기존에 있는 객체를 가져와 주입하기 때문에 결합도가 낮아지지만, A 에서 직접 생성하게 되면 의존성 관리도 A 객체가 직접 해야하기 때문에
결합도가 높아진다.

강한 결합의 예시.

```java
public class UnderstandDI {
    public static void main(String[] args) {

    }

    // 강한 결합
    public static void getDate() {
        Date date = new Date();
        System.out.println(date);
    }

    // 약한 결합
    public static void getDate(Date d) {
        Date date = d;
        System.out.println(date);
    }

    // 강한 결합 : Member를 직접 생성
    public static void memberUse1() {
        Member m1 = new Member();
    }

    //약한 결합 : 생성된 것을 주입 받음
    public static void memberUse2(Member m) {
        Member m2 = m;
    }
}

class Member {
    String name;
    String nickname;
    
    public Member() {

    }
}
```

Utility 클래스는 보통 싱글톤 방식으로 사용하기 때문에 내부적으로 private 생성자를 사용.


### 싱글턴 패턴 : 최대 N개의 객체 생성을 제한하는 패턴

예시) 
    데이터베이스 커넥션 풀
    로그 라이터
    사운드 매니저
    스코어 매니저


싱글톤이 아닌경우의 예시 -> Database 객체가 계속 생성된다.

```java

public class Database {
    private String name;

    public Datebase(String name) {
        super();
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class TestPattern1 {
    public static void main(String[] args) {
        Database d1 = new Database("1");
        Database d2 = new Database("2");
        Database d3 = new Database("3");
        Database d4 = new Database("4");
        Database d5 = new Database("5");
        Database d6 = new Database("6");

        System.out.println("database use");
    }
}
```

싱글톤인 경우

```java

public class Database {
    private static Database singleton;
    private String name;

    public Database(String name) {
        super();
        this.name = name;
    }

    public static Database getInstance(String name) {
        if(singleton == null) {
            singleton = new Database(name);
        }

        return singleton;
    }

    public String getName() {
        return name;
    }
}

public class TestPattern1 {
    public static void main(String[] args) {
        Database database;

        database = Database.getInstance("첫 번째 Database");
        System.out.println("This is the " + database.getname() + " !!!");

        database = Database.getInstance("두 번째 Database");
        System.out.println("This is the " + database.getname() + " !!!");
    }
}

```

위 예시의 문제점은 생성자 생성 방식을 제한하지 못한다. getInstance() 이 함수 말고 생성자로 호출하게 되면 싱글톤이 적용이 안된다.
그래서 생성자의 접근제어지시자를 private 으로 변경한다.

```java

public class Database {
    private static Database singleton;
    private String name;

    private Database(String name) {
        super();
        this.name = name;
    }

    public static Database getInstance(String name) {
        if(singleton == null) {
            singleton = new Database(name);
        }

        return singleton;
    }

    public String getName() {
        return name;
    }
}

```

멀티쓰레드 환경의 경우 문제점 : 각 쓰레드가 Database 객체에 접근하려고 하면 모두 새롭게 생성.
왜 그럴까 for문은 엄청 빠르다. 다른 쓰레드가 도는 시점에 singleton 객체는 모두 null임.
그래서 모두 새롭게 생성하는 것임.

```java
public class TestPattern2 {
    static int nNum = 0;

    public static void main(String[] args) {
        Runnable task = () => {
            try {
                nNum++;
                database database = Database.getInstance(nNum + "번째 Database");
                System.out.println("This is the " + database.getName() + " !!!");
            } catch (Exception e) {
            }
        };
    }

    for(int i=0; i<10; i++) {
        Thread t = new Thread(task);
        t.start();
    }
}

```

멀티스레드환경에서도 안정적인 싱글톤 구현하기 : 객체 생성시에 synchonized 선언 -> 비동기 프로그래밍시 중요한 사항! setter에 해당하는 코드들은
synchronized를 붙여야한다. 그러면 이부분은 블로킹되고 순서대로 처리된다. -> 비용이 비쌈 / 병목현상이 일어남 / 블로킹됨

```java

public class Database {
    private static Database singleton;
    private String name;

    private Database(String name) {
        super();
        this.name = name;
    }

    public synchronized static Database getInstance(String name) {
        if(singleton == null) {
            singleton = new Database(name);
        }

        return singleton;
    }

    public String getName() {
        return name;
    }
}

```

싱글톤 코드의 비용을 줄여보자.

```java
        if(singleton == null) {
            singleton = new Database(name);
        }
```

이 코드가 처음에 싱글톤 객체가 생성될때에만 유효하고 그 이후부터는 쓸데없는 코드. 생각보다 낭비가 많은 코드이다. 이거 수정해보자.

```java
public class Database {
    private static database singleton = new Datebase("products");
    private String name;

    private Database(String name) {
        try {
            Thread.sleep(100);
            this.name = name;
        } catch (Exception e) {
        }
    }

    public static Database getInstance(String name) {
        return singleton;
    }

    public String getName() {
        return name;
    }
}

```

synchronized 개선, if(singleton == null) 코드 개선