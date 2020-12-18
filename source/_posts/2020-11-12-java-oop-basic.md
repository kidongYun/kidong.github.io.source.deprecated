---
layout: post
title:  "Java 기반의 객체지향 프로그래밍에 대한 고찰"
date:   2020-11-12 11:38:54 +0900
categories: java
---

### 클래스와 인스턴스의 개념

객체지향 프로그래밍 관점에서 이 세상에 존재하는 모든 것들은 속성와 동작으로 이루어져 있다고 한다. 속성은 특정 사물이 지니고 있는 고유의 정보를 의미하고 동작이란 특정 사물들이 주체로써 행하는 모든 것들을 의미한다. 한 예시로 사람이라는 객체가 있다고 하면 이 사람의 이름, 키, 나이와 같은 정보들은 이 객체의 속성이 될 수 있으며, 이 사람이 밥을 먹고, 걷고, 말을 하는 행위들은 동작으로 볼 수 있다.

```
객체 = 속성 + 동작
```

객체지향 프로그래밍 언어에서는 이 객체라는 것을 그려내기 위해 클래스, 인스턴스라는 두 가지 개념이 존재한다. 클래스는 객체의 필수 구성 요소인 속성과 동작을 활용해 프로그래밍 세계에서 객체를 만들기 위한 도구이며, 'class' 키워드를 활용해 나타낼 수 있다. 클래스에서 사용하는 용어가 실제 객체와는 약간 다른데 주로 속성은 필드라는 이름으로 불리고 동작은 메서드라는 이름으로 부른다. 개념 자체는 동일하다.

```
클래스 = 필드 + 메서드
```

이 클래스를 통해서 만들어진 실제 객체를 프로그래밍 세계에서는 인스턴스라고 부른다. 인스턴스는 'new' 키워드를 통해서 생성되어진다. 'new' 키워드를 활용해서 동일한 클래스를 여러번 호출하면 동일한 객체가 여러 개 생성될 수 있다.

```java

/* 'class' 키워드를 통해서 객체를 표현할 수 있다. */
class Person {
    /* 필드 */
    String name;
    float height;
    int age;

    /* 메서드 */
    void eat();
    void walk();
    String talk() {
        return "제 이름은 " + name + " 이고, 키는 " + height + " 이고, 나이는 " + age + " 입니다. ";
    };
}

class Main {
    public static void main(String[] args) {
        /* 'new' 키워드를 통해서 인스턴스를 생성한다. */
        Person person = new Person();

        person.name = "john";
        person.height = 180.2;
        person.age = 31;

        System.out.println(person.talk());
    }
}

```

### 패키지의 개념

기본적으로 패키지는 파일 시스템의 디렉토리의 개념과 유사하다고 보면 된다. 한 디렉토리 내에서는 동일한 이름을 가진 파일을 중복 생성할 수 없고 다른 디렉토리에서는 가능하듯이 자바파일도 한 패키지에서는 중복되는 이름을 가진 파일을 만들 수 없지만. 다른 패키지에서는 가능하다. 실무의 관점에서 패키지 경로에는 보통 특정 소스코드를 개발한 회사, 해당 프로젝트의 이름 등이 들어가는데. 이를 통해 각 코드의 출처를 구분지을 수 있다.

```java

package team1;

public class Circle {
    final double PI = 3.14;
    double rad;

    public void setRad(double r) {
        rad = r;
    }

    // 원의 넓이 반환
    public double getArea() {
        return (rad * rad) * PI;
    }
}

package team2;

public class Circle {
    final double PI = 3.14;
    double rad;

    public void setRad(double r) {
        rad = r;
    }

    // 원의 둘레 반환
    public double getPerimeter() {
        return (rad * 2) * PI;
    }
}

```

동일하게 Circle.java 파일을 각각 team1, team2 패키지에 만들었다. 다른 패키지에 생성했음으로 이 둘다 정상 작동할 것이다.

```java

public class A_CircleUsing {
    public static void main(String[] args) {
        team1.Circle c1 = new team1.Circle();
        c1.setRad(3.5);
        System.out.println("반지름 3.5 원 넓이: " + c1.getArea());

        team2.Circle c2 = new team2.Circle();
        c2.setRad(3.5);
        System.out.println("반지름 3.5 원 둘레: " + c2.getPerimeter());
    }
}


import team1.Circle;
import team2.Circle;

public class A_CircleUsing {
    public static void main(String[] args) {
        Circle c1 = new team1.Circle();
        c1.setRad(3.5);
        System.out.println("반지름 3.5 원 넓이: " + c1.getArea());

        Circle c2 = new team2.Circle();
        c2.setRad(3.5);
        System.out.println("반지름 3.5 원 둘레: " + c2.getPerimeter());
    }
}

```

사용하는 자바파일에서 중복되는 이름이 없으면 'import' 키워드를 사용해서 패키지 경로를 생략할 수 있지만. 중복되는 경우는 사용하는 자바가 어떤 Circle을 요구하는지 인지하지 못 함으로 패키지 경로까지 입력해줘야 한다.

### 오버로딩의 개념

오버로딩이란 하나의 클래스 내부에 함수의 시그니처는 다르지만 함수의 이름이 같은 메서드를 여러 개 정의하는 방법이다.
아래는 덧셈의 기능을 가지고 있는 계산기 객체인데 덧셈 시 들어로는 파라미터의 타입 별로 add() 함수를 오버로딩 하였다.

```java

class Calc {
    int add(int a, int b) {
        return a + b;
    }

    int add(int a) {
        return a + 1;
    }

    double add(double a, double b) {
        return a + b;
    }
}

public class calculation {
    public static void main(String[] args) {
        Calc calc = new Calc();

        calc.add(3, 9);
        calc.add(3);
        calc.add(3.0, 9.0);
    }
}

```

### 생성자의 개념

생성자란 인스턴스 생성과 함께 자동적으로 호출되는 특수한 메서드. 'new' 키워드를 통해 인스턴스를 만들게 되면 필요한 전처리 작업을 이 곳에서 수행할 수 있다. 생성자를 명시하지 않는 경우에는 인수가 없는 디폴트 생성자가 자동으로 만들어진다.

생성자는 일반 함수와 조금은 다른 특성을 가지고 있는데 우선 메서드와 같은 모양이지만 반환형이 없고 클래스의 이름과 동일한 이름을 가진다.

개발자가 매개 변수가 잇는 생성자를 만든 경우 아무런 파라미터가 없는 디폴트 생성자를 호출하면 에러가 발생한다.
이 경우 매개 변수가 없는 디폴트 생성자를 호출하기 위해서는 디폴트 생성자도 같이 구현해 주어야 한다.

```java

package step1;

class Book {
    String title;
    int price;
    int num;

    Book() {
        title = "자바 클래스 기초";
        price = 10000;
    }

    Book(String title, int price) {
        this.title = title;
        this.price = price;
    }

    void print() {
        System.out.println("제목 : " + title);
        System.out.println("가격 : " + price);
        System.out.println("주문수량 : " + num);
        System.out.println("합계금액 : " + price * num);
    }
}

public class MyBook {
    public static void main(String[] args) {
        Book book = new Book();
        Book book = new Book("자바 디자인 패턴", 20000);
        book.num = 5;
        book.print();
    }
}

```

### 메모리 모델의 개념
<<<<<<< HEAD

JVM도 하나의 응용프로그램으로 동작하는 것으로 운영체제로부터 할당받은 메모리 공간을 기반으로 자바 프로그램을 실행한다.
JVM은 운영체제로부터 할당받은 메모리 공간을 이용해서 자기 자신도 실행을 하고, 자바 프로그램도 실행을 하는데 이 메모리 공간을 이용할 때 공간 활용의 효율성을 높이기 위해서 메서드영역, 스택영역, 힙영역으로 구분하여 사용한다.

바이트코드 : 고급언어로 작성된 소스 코드를 가상머신이 이해할 수 있는 중간 코드로 컴팡리 된것을 말한다.
가상머신은 이 바이트코드를 각각의 하드웨어 아키텍처에 맞는 기계어로 번역.

메서드영역 : 자바 바이트코드는 JVM이 구분하는 메모리 공간 중에서 메서드 영역에 저장된다.
static 으로 선언된 클래스 변수도 메서드 영역에 저장된다. 이 영역에 저장된 내용은 프로그램 시작 전에 로드되고 프로그램 종료 시 소멸 된다.

스택 영역 : 매개변수와 지역변수가 할당되는 메모리 공간. 프로그램이 실행되는 도중에 임시로 할당되었다가 바로 이어서 소멸되는 특징이 있는 변수가 할당 된다. 이 영역에 저장되는 변수는 해당 변수가 선언된 메서드 종료 시 소멸된다.

힙 영역 : 사실 자바에서 가장 유별나고, 중요하게 알고있어야 하는 메모리 모델은 바로 이 영역이다. 위에서 'new' 연산을 통해서 인스턴스를 생성할 수 있음을 배웠는데, 그러면 이 생성된 인스턴스는 어느 시점에 사라질까?. 당연하게도
자바라는 언어도 컴퓨터의 자원을 활용하기 때문에 항상 인스턴스를 생성만 하고 제거를 하지 않는다면 메모리 영역이 꽉 차서 문제가 생길 것이다. 자바에는 가비지 컬렉터라는 요소가 존재한다. 이 녀석은 주기적으로 당신이 만든 프로그램을
점검하며 사용하지 않는 객체가 있는지 없는지 확인한다. 그리고 사용하지 않는 객체라고 확신이 들면 이를 소멸시킨다.

그렇다면 가비지 컬렉터가 사용하지 않는 객체라고 확신하는 기준은 무엇일까. 일반적으로 참조변수에 의한 참조가 전혀 이뤄지지 않는 인스턴스가 소명의 대상이 된다. 참조관계가 끊어진 인스턴스는 유저가 접근할 방법이 없기 때문이다.

가비지 컬렉터의 발생 주기는 개발자가 확신을 할수 없다. JVM 내부적으로 임의의 동작에 의해서 발현되기 때문이다. 그렇기 때문에 특정 시점에 반드시 객체를 소멸시켜야 할때도 있는데 그럴 때는 아래와 같은 코드를 짜서 소멸시킬 수 있다.

=======

JVM도 하나의 응용프로그램으로 동작하는 것으로 운영체제로부터 할당받은 메모리 공간을 기반으로 자바 프로그램을 실행한다.
JVM은 운영체제로부터 할당받은 메모리 공간을 이용해서 자기 자신도 실행을 하고, 자바 프로그램도 실행을 하는데 이 메모리 공간을 이용할 때 공간 활용의 효율성을 높이기 위해서 메서드영역, 스택영역, 힙영역으로 구분하여 사용한다.

바이트코드 : 고급언어로 작성된 소스 코드를 가상머신이 이해할 수 있는 중간 코드로 컴팡리 된것을 말한다.
가상머신은 이 바이트코드를 각각의 하드웨어 아키텍처에 맞는 기계어로 번역.

메서드영역 : 자바 바이트코드는 JVM이 구분하는 메모리 공간 중에서 메서드 영역에 저장된다.
static 으로 선언된 클래스 변수도 메서드 영역에 저장된다. 이 영역에 저장된 내용은 프로그램 시작 전에 로드되고 프로그램 종료 시 소멸 된다.

스택 영역 : 매개변수와 지역변수가 할당되는 메모리 공간. 프로그램이 실행되는 도중에 임시로 할당되었다가 바로 이어서 소멸되는 특징이 있는 변수가 할당 된다. 이 영역에 저장되는 변수는 해당 변수가 선언된 메서드 종료 시 소멸된다.

힙 영역 : 사실 자바에서 가장 유별나고, 중요하게 알고있어야 하는 메모리 모델은 바로 이 영역이다. 위에서 'new' 연산을 통해서 인스턴스를 생성할 수 있음을 배웠는데, 그러면 이 생성된 인스턴스는 어느 시점에 사라질까?. 당연하게도
자바라는 언어도 컴퓨터의 자원을 활용하기 때문에 항상 인스턴스를 생성만 하고 제거를 하지 않는다면 메모리 영역이 꽉 차서 문제가 생길 것이다. 자바에는 가비지 컬렉터라는 요소가 존재한다. 이 녀석은 주기적으로 당신이 만든 프로그램을
점검하며 사용하지 않는 객체가 있는지 없는지 확인한다. 그리고 사용하지 않는 객체라고 확신이 들면 이를 소멸시킨다.

그렇다면 가비지 컬렉터가 사용하지 않는 객체라고 확신하는 기준은 무엇일까. 일반적으로 참조변수에 의한 참조가 전혀 이뤄지지 않는 인스턴스가 소명의 대상이 된다. 참조관계가 끊어진 인스턴스는 유저가 접근할 방법이 없기 때문이다.

가비지 컬렉터의 발생 주기는 개발자가 확신을 할수 없다. JVM 내부적으로 임의의 동작에 의해서 발현되기 때문이다. 그렇기 때문에 특정 시점에 반드시 객체를 소멸시켜야 할때도 있는데 그럴 때는 아래와 같은 코드를 짜서 소멸시킬 수 있다.

>>>>>>> 9b6833358e0102e57e17e999360fbe7ac9a03809
```java

/* 가비지 컬렉터를 호출함 */
System.gc();

/* 가비지 컬렉터에 의해서 소멸이 결정된 인스턴스를 즉시 소멸 */
System.runFinalization();

```


### 접근제어지시자의 개념

실무의 관점에서 접근제어지시자는 중요한 키워드이다. 이 것을 활용해 내가 만든 이 코드를 누구에게까지 사용하도록 할 것인지를 결정할 수 있다. 이 접근제어지시자는 클래스 앞에 사용되었을 떄와 필드, 메서드 앞에 사용되었을 때는 약간 다름으로 구분해서 확인해보자.

#### 클래스에 지정되었을 때

public 으로 선언하게 되면 이 클래스는 어느 곳에서든 인스턴스 생성이 가능하다. 그리고 아무런 접근제어시시자를 주지 않는 것을 default 라고 하는데 default 는 동일 패키지로 묶인 클래스 내에서만 인스턴스 생성을 허용한다.

#### 인스턴스 멤버에 지정했을 때

```

지시자      클래스내부      동일패키지      상속받은클래스      이외의영역
private     O               X               X                   X
default     O               O               X                   x
protected   O               O               O                   X
public      O               O               O                   O

```

```java

package team1;

public class AnotherClass1 {
    public int num1;
    private int num2;
    protected int num3;
    int num4;

    public void test1() { System.out.println("test1"); }
    private void test1() { System.out.println("test2"); }
    protected void test1() { System.out.println("test3"); }
    void test1() { System.out.println("test4"); }
}

```

```java

package team2;

public class AnotherClass2 {
    public int num1;
    private int num2;
    protected int num3;
    int num4;

    public void test1() { System.out.println("test1"); }
    private void test1() { System.out.println("test2"); }
    protected void test1() { System.out.println("test3"); }
    void test1() { System.out.println("test4"); }
}

```

```java

public class AccessTest {
        public int num1;
    private int num2;
    protected int num3;
    int num4;

    public void test1() { System.out.println("test1"); }
    private void test1() { System.out.println("test2"); }
    protected void test1() { System.out.println("test3"); }
    void test1() { System.out.println("test4"); }
    
    public static void main(String[] args) {
        /* 같은 클래스이기 때문에 모두 접근이 가능함 */
        AccessTest at = new AccessTest();
        at.num1 = 1;
        at.num2 = 2;
        at.num3 = 3;
        at.num4 = 4;

        AnotherClass1 ac1 = new AnotherClass1();
        ac1.num1 = 1;
        ac1.num2 = 2;   // private 이기 때문에 접근 불가능
        ac1.num3 = 3;
        ac1.num4 = 4;

        AnotherClass2 ac2 = new AnotherClass2(); 
        ac2.num1 = 1;
        ac2.num2 = 2;   // private 접근 불가능.
        ac2.num3 = 3;   // 다른 패키지 이기 때문에 접근 불가능.
        ac2.num4 = 4;   // 다른 패키지 이기 때문에 접근 불가능.
    }
}

```


### 상속의 개념

객체지향에서 상속이라 함은 상위클래스의 모든 것이 하위클래스에게 전달되는 것을 뜻한다. 그러나 상위클래스의 멤버변수와 멤버함수 중, private 으로 접근제한이 된 경우에는 하위클래스로 전달이 되지 않는다. 상속의 활용성, 그 의미는 여기서 다루기에는 사실 쉽지 않다. 다음에 다른 글에서 확인하도록 하고 여기서는 문법만 확인하도록 하자. 상속받은 클래스 즉 서브클래스를 만들기 위해서는 'extends'를 사용한다. 자바에서 여러 개의 클래스를 동시에 상속하는 다중 상속은 허용되지 않는다.

```java

class Book {
    String title;
    
    void printBook() {
        System.out.println("제목: " + title);
    }
}

class Novel extends Book {
    String writer;
    
    void printNov() {
        printBook();
        System.out.println("저자: " + writer);
    }
}

class Magazine extends Book {
    int day;

    void printMag() {
        printBook();
        System.out.println("발매일: " + day + "일");
    }
}

public class Booshelf {
    public static void main(String[] args) {
        Novel nov = new Novel();
        nov.title = "홍길돌전";
        nov.writer = "허균";

        Magazine mag = new Magazine();
        mag.title = "월간 자바";
        mag.day = 20;

        nov.printNov();
        System.out.println();
        mag.printMag();
    }
}

```

### 오버라이딩의 개념

오버라이딩이란 부모 메서드와 동일한 이름, 동일한 인수를 가지는 메서드를 정의하여 메서드를 덮어쓰는 것이다. 반환값의 형도 같아야만 한다.
오버라이딩은 하위클래스에서 상속 받은 메서드를 단순 재사용 하지 않고 재정의하여 기능을 변경하거나 새로운 기능을 추가를 하고 싶을 때 사용한다. 즉 하위클래스에서 상위클래스의 특정 메서드를 다시 정의하는 것이다.
오버라이딩은 많은 객체지향 디자인 패턴에 매우 자주 사용되는 기법이다. 오버라이딩은 추상 클래스와 합쳐져서 객체지향 방법론에서 장점으로 많이 거론되는 확장성을 실현하는 데 많은 도움을 주게 되므로, 이 개념은 필수적이다.

```java

class Animal {
    String name;
    int age;

    void printPet() {
        System.out.println("이름 : " + name);
        System.out.println("나이 : " + age);
    }
}

class Dog extends Animal {
    String variety;

    void printPet() {
        super.printPet();
        System.out.println("종류 : " + variety);
    }
}

public class Pet {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.name = "진돌이";
        dog.age = 5;
        dog.variety = "진돗개"
        dong.printPet();
    }
}

```

<<<<<<< HEAD
### 추상 클래스의 개념

추상 클래스는 추상메서드를 한 개라도 가지고 있으면 추상 클래스라고 한다. 그렇다면 추상 클래스는 뭘까? 함수의 실제 동작이 되는 내부 과정은 없고 함수 시그니처만을 정의한 메서드를 추상 메서드라고 한다. 추상 메서드와 추상 클래스는 abstract 키워드를 사용하여 다음과 같이 정의할 수 있다. 추가적으로 추상클래스를 상속하는 클래스는 추상메서드를 반드시 오버라이딩해서 구현해야 한다.
=======
### 추상클래스의 개념

처리내용을 기술하지 않고 호출하는 방법만을 정의한 메서드를 추상 메서드라고 한다. 추상 메서드를 한개라도 가진 클래스를 추상 클래스라 한다. 추상메서드, 추상 클래스 abstract 라는 수식자를 사용하여 다음과 같이 정의.
추상클래스를 상속하는 클래스는 추상메서드를 반드시 오버라이딩해서 구현해야한다. 추상클래스를 사용하는 이유 : 고객이 말하지 않아도 추상클래슨느 무조건 구현해야 하기 때문에 까먹고 안 만들 일도 없다. 이건 아닌거같은데
>>>>>>> 9b6833358e0102e57e17e999360fbe7ac9a03809

```java

abstract class Animal {
    int age;
    abstract void cry();
}

class Dog extends Animal {
    void cry() {
        System.out.println("멍멍");
    }
}

class Cat extends Animal {
    void cry() {
        System.out.println("야옹");
    }
}

public class AbstractClassEx {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.cry();

        Cat cat = new Cat();
        cat.cry();
    }
}

```

### 인터페이스의 개념

인터페이스는 함수 시그니처만 가지고 있는 추상 클래스라고 생각하면 쉽다.(자바 버전이 올라가면서 인터페이스도 구현체, 필드를 가질 수 있게 되었다. 여기서는 전통적인 인터페이스의 개념이다) 이렇게 모두 선언만 되어있고 실제로 구현된 것은 없기 때문에 이를 구현하는 서브 클래스들은 인터페이스에 선언되어 있는 함수들을 모두 구현하여야 한다. 클래스에서 인터페이스를 이용하도록 하게 하는 것을 '인터페이스의 구현'이라고 하고 이를 위해서는 'implements' 키워드를 사용한다.
인터페이스는 다중상속이 가능하다. 인터페이스는 복수의 인터페이스를 상속하여 새로운 인터페이스를 만들수 있다.

```java

interface Greet {
    void greet();
}

interface Bye {
    void bye();
}

class Morning implements Greet, Bye {
    public void bye() {
        System.out.println("안녕히 계세요.");
    }

    public void greet() {
        System.out.println("안녕하세요.");
    }
}

public class Meet {
    public static void main(String[] args) {
            Morning morning = new Morning();
            morning.greet();
            morning.bye();
    }
}

```

```java

interface Greet {
    void greet();
}

interface Bye extends Greet {
    void bye();
}

class Morning implements Bye {
    public void bye() {
        System.out.println("안녕히 계세요.");
    }

    public void greet() {
        System.out.println("안녕하세요.");
    }
}

public class Meet {
    public static void main(String[] args) {
        Morning morning = new Morning();
        morning.greet();
        morning.bye();
    }
}

```

### 다형성

상속한 클래스의 오브젝트는 슈퍼 클래스로도 서브 클래스로도 다룰 수 있다. 이렇게 하나의 오브젝트와 메서드가 많은 형태를 가지고 있는 것을 다형성이라고 한다. 슈퍼 클래스의 오브젝트 생성 서브 클래스의 오브젝트는 슈퍼 클래스의 오브젝트에 대입할 수 있다. 상위 클래스의 객체를 하위 클래스의 객체로 대입할 수는 없다. Sub 클래스의 설계도를 바탕을 name 변수를 사용하면, 생성된 객체에는 해당 변수가 없어 에러가 발생하게 된다. 이러한 이유로 상위 클래스의 객체를 하위 클래스의 객체로 대입하는 것을 막고 있다.

```java

abstract class Calc {
    int a = 5;
    int b = 6;

    abstract void plus();
}

class Mycalc extends Calc {
    void plus() { System.out.println(a + b); }
    void minus() { System.out.println(a - b); }
}

public class Polymorphism1 {
    public static void main(String[] args) {
        Mycalc mycalc1 = new MyCalc();
        myCalc1.plus();
        myCalc1.minus();

        Calc myCalc2 = new MyCalc();
        myCalc2.plus();
        myCalc2.minus();    // 에러


    }
}

```

```java

interface Printable {
    void print(String doc);
}

class PrnDrvSamsung implements Printable {
    public void print(String doc) {
        System.out.println(doc + "\nFrom Samsung : Black-White Ver");
    }
}

class PrnDrvEpson implements printable {
    public void print(String doc) {
        System.out.println(doc + "\nFrom Epson : Black-White Ver");
    }
}

public class Polymoriphism2 {
    public static void main(String[] args) {
        String doc = "프린터로 출력을 합니다";

        Printable prn1 = new PrnDrvSamsung();
        prn1.print(doc);

        Printable prn2 = new PrnDrvEpson();
        prn2.print(doc);
    }
}

```

어있고 이를 스택영역의 참조변수가 가리키고 있다. 슈퍼타입의 참조변수가 이 힙 영역의 서브 타입의 인스턴스를 가리키고 다형성을 이용한 클래스 간의 형 변환, 서브 클래스의 오브젝트는 슈퍼 클래스의 오브젝트에 대입할 수 있다. 힙 영역에 실제 인스턴스가 저장 되 있을 때 슈퍼 타입을 서브 타입의 참조변수로 형변환하고 이 서브타입의 참조변수로 같은 인스턴스를 가리키게 하면 단순히 인스턴스는 그대로이고 참조변수의 형만 변환되는거기 때문에 서브클래스만의 필드들을 접근할 수 있다.

```java

class PBoard { }

class CBoard extends PBoard { }

public class ClassCast {
    public static void main(String[] args) {
        PBoard sbd1 = new CBoard();
        CBoard sbd2 = (CBoard) sbd1;

        System.out.println("------------------------");

        PBoard ebd11 = new PBoard();
        CBoard ebd2 = (CBoard) ebd1;
        /* 이건 슈퍼클래스 인스턴스를 서브클래스의 참조변수로 가리키려고 하기때문에 오류. */
    }
}

```

### 은닉화의 개념

객체의 변수를 public 으로 설정하면 외부에서 마음대로 이 변수를 사용할 수 있다. 의도하지 않은 범위의 값을 넣을 수 있다. 원하지 않는 데이터타입을 강제적으로 형변환하여 넣을 수도 있다. 원하지 않는 데이터 넣는것을 방지하기 위해 Java Beans 패턴 (Getter / Setter) 를 사용.

```java

class SimpleBox {
    private int num;

    SimpleBox(int num) {
        this.num = num;
    }

    public int getNum() {
        return num;
    }

    public void setnum(int num) {
        this.num = num;
    }
}

public class ThisUseEx {
    public static void main(String[] args) {
        SimpleBox box = new SimpleBox(5);
        box.setNum(10);
        System.out.println(box.getNum());
    }
}

```

### instanceof 연산자

'instanceof'는 오브젝트가 지정한 클래스의 오브젝트인지를 조사하기 위한 연산자이다. 왼쪽 오브젝트가 오른쪽 클래스 또는 서브 클래스의 오브젝트라면 true. 'instanceof'는 지정한 인터페이스를 오브젝트가 구현하고 있는지를 조사할 수도 있다. 왼쪽 오브젝트가 오른쪽 인터페이스를 구현하고 있으면 true.

```java

interface Cry {
    void cry();
}

class Cat implements Cry {
    public void cry() {
        System.out.println("야옹");
    }
}

class Dog implements Cry {
    public void cry() {
        system.out.println("멍멍");
    }
}

public class CheckCry {
    public static void main(String[] args) {
        Cry test1 = new Cat();
        Cry test1 = new Dog();

        if(test1 instanceof Cat) {
            test1.cry();
        } else {
            System.out.println("고양이가 아닙니다.");
        }
    }
}
```

### Class 클래스

자바의 모든 클래스와 인터페이스는 컴파일 후 class 파일로 생성된다. class 파일에는 객체의 정보(멤버변수, 메서드, 생성자 등)가 포함되어 있음
Class 클래스는 컴파일된 class 파일에서 객체의 정보를 가져올 수 있음.
reflection 프로그래밍 : Class 클래스르 ㄹ이용하여 클래스의 정보를 가져오고 이를 활용하여 인스턴스를 생성하고, 메서드를 호출하는 등의 프로그래밍 방식

Class.forName() 메서드로 동적 로딩하기

프로그래밍 할 때는 어떤 클래스를 사용할지 모를때 변수로 처리하고, 실행될 때 해당 변수에 대입된 값의 클래스가 실행될 수 있도록 Class 클래스에서 제공하는 static 메서드

동적로딩이란 -> 컴파일 시에 데이터타입이 모두 바인딩되어 자료형이 로딩되는 것이 아니라 실행중에 데이터 타입을 알고 바인딩 되는 방식
컴파일 타임에 체크 할 수 없음으로 해당 문자열에 댛나 클래스가 없는 경우 예외(ClassNotFoundException)이 발생할 수 있다.

외부에서 소스코드 없이 클래스만 제공받아 사용할 경우에도 많이 사용된다.

```java

public class MyBook {
    private String title;
    public String author;

    public MyBook(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}

public class ClassForNameTest {
    public static void main(String[] args) throws ClassNotFoundException {
        Class strClass = Class.forName("MyBook");

        Constructor[] cons = strClass.getConstructors();
        for(Constructor c : cons) {
            System.out.println(c);
        }

        Field[] fields = strClass.getFields();
        for(Field f : fields) {
            System.out.println(f);
        }

        Method[] methods = strClass.getmethods();
        for(Method m : methods) {
            System.out.println(m);
        }
    }
}
```


### 절차지향 및 객체지향의 개념

절차지향과 객체지향 개념은 반대되는 개념이 아니며, 서로가 보완 될 수 있는 개념이다. 자동차를 만드는 과정을 절차지향적 관점과 객체지향적 관점 두 부분으로 나누어 생각해보자. 자동차를 만들기 위해서는 차체, 바퀴,엔진, 핸들 ,의자, 엑셀 등등 많은 부품이들이 있어야 한다. 절차지향 프로그래밍의 경우 '차체 -> 바퀴 -> 엔진 -> 핸들' 와 같이 작업 순서가 존재한다. 트랜잭션 처리와 같이 순차적인 처리가 중요시 되는 곳에서는 이 절차지향 프로그래밍 방법을 사용해야 한다.
이들은 서로 분리되면 안되고 함수 호출 순서가 틀려서도 안되며 하나가 고장나면 전체 기능이 마비된다.
절차지향은 기본적으로 데이터를 중심으로 함수를 구성한다. 함수의 호출 순서가 바뀌면 데이터의 전달과 값이 변할 수 있다.

객체지향의 프로그래밍의 경우는 공장에서 각 공장에서 각 요소들의 제품들을 제작한다. 제작 순서 자체는 의미를 가지지 않을 때 활용할 수 있다. 물론 비동기 적으로 동작할수도 있고 아닐 수도 있다. 객체지향 프로그래밍은 개발하려는 것을 기능별로 묶어 모듈화를 하고 모듈을 재활용하는 프로그래밍 방식이다. 순서가 있을 수도 있지만 이는 고려의 대상은 아니다. 이들은 각각 따로 독립적으로 개발되어 나중에 한곳에 모여 자신의 기능만 제대로 발휘하면 된다. 부품들이 결합되어 움직이다 어느 하나가 고장이 나면 고장난 부품만 고쳐주면 되고 다른 부품들은 영향을 받지 않는다. 필요한 부품을 다른 것으로 교체할 수 있다.
