---
layout: post
title:  "Java O.O.P Class"
date:   2020-11-12 11:38:54 +0900
categories: java
---

### 클래스의 개념 (1)

클래스란 간단히 말하면 객체와 관련된 데이터와 처리동작을 한데 모은 것이다.

Class is the blueprint of the object. This blueprint should be consist of data and method. so You can't use the class directly.

You should create the object using 'new' keyword

```java
    /* 클래스 정의*/
    class Npc {
        /* 필드 - 데이터 */
        String name;
        int hp;

        /* 메서드 - 동작 */
        void say() {
            System.out.println("안녕하세요");
        }
    }

    public class MyNpc {
        public static void main(String[] args) {
            Npc saram1 = new Npc();

            saram1.name = "경비";
            saram1.hp = "100";

            System.out.println(saram1.name + ":" + saram1.hp);

            saram1.say();
        }
    }
```

### 클래스의 개념 (2)

메서드만 있는 클래스
```java

class Calc {
    int add(int a, int b) {
        return a + b;
    }
}

public class Calculation {
    public static void main(String[] args) {
        // 객체 생성
        Calc calc = new Calc();
        // 메서드 호출
        int nReturn = calc.add(3, 9);

        System.out.println("3 + 9 = " + nReturn);
    }
}

```

필드만 있는 클래스
```java

class Book {
    String title;
    String author;
    int price;
}

public class MyBook {
    public static void main(String[] args) {
        // 객체 생성
        Book book = new Book();
        // 필드 접근
        book.title = "클래스의 기초";
        book.author = "홍길동";
        book.price = 10000;

        System.out.println(book.title + ":" + book.author + ":" + book.price);
    }
}
```

### 패키지의 개념

디렉토리명과 동일하다고 보면된다.

공간적/ 접근적 충돌 해결을 위한 패키지 선언.

클래스 접근 방법의 구분
- 서로 다른 패키지의 두 클래스는 인스턴스 생성 시 사용하는 이름이 다르다.

클래스의 공간적인 구분
- 서로 다른 패키지의 두 클래스 파일은 저장되는 위치가 다르다.


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

사용하는 방법

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

```

import를 사용해도 된다.

### Overloading

오버로딩이란 하나의 클래스 내에 인수의 개수나 형이 다른 동일한 이름의 메서드를 여러개 정의하는 것이다.

함수의 형태 = 시그니처.

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

public class Calculation {

}

public class calculation {
    public static void main(String[] args) {
        Calc calc = new Calc();

        int nReturn1 = calc.add(3, 9);
        int nReturn2 = calc.add(3);
        double nReturn3 = calc.add(3.0, 9.0);
    }
}

```

### 생성자

생성자란 오브젝트 생성과 함께 자동적으로 호출되는 특수한 메서드.
그러므로 개발자가 생성자를 기술하지 않는 경우, 인수가 없는 생성자가 자동으로 만들어진다. = 디폴트 생성자.

디폴트 생성자의 특징 :
메서드와 같은 모양이지만 반환형이 없다.
클래스의 이름과 동일한 이름을 가진다.

생성자는 가장 흔한 오버로딩의 대상.

개발자가 매개 변수가 잇는 생성자를 만든 경우, 디폴트 생성자를 호출하면 에러가 발생한다.

이 경우 매개 변수가 없는 디폴트 생성자를 호출하기 위해서는 디폴트 생성자도 같이 구현해 주어야 한다.

복제 생성자.
동일한 클래스의 오브젝트를 인수로 받아서, 대응하는 필드에 값을 대입하는 생성자를 복제 생성자라 한다.

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

복제생성자

```java

class Book {
    String title;
    int price;

    Book(String title, int price) {
        this.title = title;
        this.price = price;
    }

    Book(Book copy) {
        title = copy.title;
        price = copy.price;
    }

    void print() {
        System.out.println("제목 : " + title);
        System.out.println("가격 : " + price);
    }
}
public class MyBook {
    public static void main(String[] args) {
        Book book1 = new Book();
        Book book1 = new Book("자바 클래스 기초", 10000);
        book1.print();


        Book book2 = new Book(book1);
        book2.title = "자바 디자인 패턴";
        book2.print();
    }
}

```

### 메모리 모델

JVM은 운영체제로부터 할당받은 메모리 공간을 기반으로 자바 프로그램을 실행해야 한다.

JVM은 운영체제로부터 할당받은 메모리 공간을 이용해서 자기 자신도 실행을 하고, 자바 프로그램도 실행을 한다.

자바가상머신의 메모리 모델
메모리 공간 활용의 효율성을 높이기 위해 메서드영역, 스택영역, 힙영역으로 구분하여 사용.

바이트코드 -> 고급언어로 작성된 소스 코드를 가상머신이 이해할 수 있는 중간 코드로 컴팡리 된것을 말한다.
가상머신은 이 바이트코드를 각각의 하드웨어 아키텍처에 맞는 기계어로 번역.

메서드영역 -> 자바 바이트코드는 JVM이 구분하는 메모리 공간 중에서 메서드 영역에 저장된다.
static 으로 선언된 클래스 변수도 메서드 영역에 저장된다.

이 영역에 저장된 내용은 프로그램 시작 전에 로드되고 프로그램 종료 시 소멸된다.

스택 영역 -> 매개변수, 지역변수가 할당되는 메모리 공간.
프로그램이 실행되는 도중에 임시로 할당되었다가 바로 이어서 소멸되는 특징이 있는 변수가 할당도니다.
이 ㅕㅇ역에 저장도니 변수는 해당 변수가 선언된 메서드 종료 시 소멸된다.

지역변수는 스택에 할당된다.
스택에 할당된 지역변수는 해당 메서드를 빠져 나가면 소멸된다.
할당 및 소멸의 특성상 그 형태가 장작을 쌓는 것과 유사해서 스택이라 이름이 지어 졌다.
할당 및 소멸의 특성상 메서드 별 스택이 구분이 된다.

힙 영역 -> 이녀석만 GC가 개입하는거였네 와 몰랐다.
인스턴스(객체)가 생성되는 메모리 공간.
JVM에 의한 메모리 공간의 정리 GC가 이뤄지는 공간
할당은 프로그래머가 소멸은 JVM이 처리한다.
참조변수에 의한 참조가 전혀 이뤄지지 않는 인스턴스가 소멸의 대상이 도니다.
따라서 JVM은 인스턴스의 참조관계를 확인하고 소멸할 대상을 선정한다.

참조관계가 끊어진 인스턴스는 유저가 접근을 할수 없음으로 그래서 가비지 컬렉션의 대상이 됨.

GC는 한번도 발생하지 않을 수 있다.
GC가 발생하면 소멸의 대상이 되는 인스턴스는 결정이되지만 이것이 실제 소멸ㅇ로 바로 이뤄지는것은 아니다.
인스턴스의 실제 소멸로 이어지지 않ㅇ느 상태에서 프로그램이 종료가 될수 있다.
종료가 되면 어차피 인스턴스는 소멸된다.

System.gc(); -> GC를 호출함.
System.runFinalization(); GC에 의해서 소멸이 결정된 인스턴스를 즉시 소멸.


### 접근제어지시자.

#### 클래스에 지정되었을 때

public 어디서든 인스턴스 생성이 가능하다.

default 동일 패키지로 묶인 클래스 내에서만 인스턴스 생성을 허용한다. -> 와 이것도 몰랐음. ㅋㅋ

#### 인스턴스 멤버

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


### 상속

객체지향에서 상속이라 함은 상위클래스의 모든 것이 하위클래스에게 전달되는 것을 뜻한다.
그러나 상위클래스의 멤버변수와 멤버함수 중, private 으로 접근제한이 된 경우에는 하위클래스로 전달이 되지 않는다.

장점
재사용성 증대.
확장 용이성. (리얼로??)

서브클래스를 만들기 위해서는 extends를 사용한다.
자바에서 여러 개의 클래스를 동시에 상속하는 다중 상속은 허용되지 않는다.

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

### 오버라이딩

오버라이딩이란 상속된 메서드와 동일한 이름, 동일한 인수를 가지는 메서드를 정의하여 메서드를 덮어쓴느 것이다. 반환값의 형도 같아야만 한다.

오버라이드는 하위클래스에서 상속 받은 메서드를 단순 재사용 하지 않고 재정의하여 다른 연산을 수행하고 싶을 때 사용한다.
즉 하위클래스에서 상위클래스의 특정 메서드를 다시 정읳나ㅡㄴ 것이다.
-> 기능의 변경
-> 기능의 추가.

오버라이딩은 많은 객체지향 디장니 패턴에 매우 자주 사용되는 기법이다. 오버라이딩은 추상 클래스와 합쳐져서 객체지향 방법론에서 장점으로 많이 거론되는 확장성을 실현하는 데 많은
도움을 주게 되므로, 이 개념은 필수적이다.

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
        super.printPet();   // -> 이게 있으면 부모클래스의 기능도 살리고 새로운 기능을 넣는거기 때문에 기능으 ㅣ추가.
                            // -> 없으면 새로운 기능을 재정의 하는거기때문에 기능의 재정의
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