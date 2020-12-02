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

추상클래스

처리내용을 기술하지 않고 호출하는 방법만을 정의한 메서드를 추상 메서드라고 한다.
추상 메서드를 한개라도 가진 클래스를 추상 클래스라 한다.

추상메서드, 추상 클래슨느 abstract 라는 수식자를 사용하여 다음과 같이 정의.

추상클래스를 상속하는 클래스는 추상메서드를 반드시 오버라이딩해서 구현해야한다.

추상클래스를 사용하는 이유 : 고객이 말하지 않아도 추상클래슨느 무조건 구현해야 하기 때문에 까먹고 안 만들 일도 없다.
이건 아닌거같은데

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

### 인터페이스

인터페이스란 상속관계가 아닌 클래스에 기능을 제공하는 구조이다.
클래스와 비슷한 구조이지만, 정의와 추상 메서드만이 멤버가 될 수 있단든 점이 다르다.
클래스에서 인터페이스를 이용하도록 하게 하는 것을 '인터페이스의 구현'이라고 한다.
인터페이스를 구현하기 위해서는 implements 를 사용한다.

인터페이스는 다중상속이 가능하다.

인터페이스는 복수의 인터페이스를 상속하여 새로운 인터페이스를 만들수 있다.

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

=======
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


### 다형성

상속한 클래스의 오브젝트는 슈퍼 클래스로도 서브 클래스로도 다울 수 있다.
이렇게 하나의 오브젝트와 메서드가 많은 형태를 가지고 있는 것을 다형성이라고 한다.

슈퍼클래스의 오브젝트 생성
서브클래스의 오브젝트는 슈퍼 클래스의 오브젝트에 대입할 수 있다.

상위클래스의 객체를 하위클래스의 객체로 대입할 수는 없다.

Sub 클래스의 설계도를 바탕을 name 변수를 사용하면, 생성된 객체에는 해당 변수가 없어 에러가 발생하게 된다.
이러한 이유로 상위클래스의 객체를 하위클래스의 객체로 대입하는 것을 막고 있다.


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


다형성을 이용한 클래스간의 형변환

서브 클래스의 오브젝트는 슈퍼 클래스의 오브젝트에 대입할 수 있다.

힙영역에 실제 인스턴스가 저장되어있고 이를 스택영역의 참조변수가 가리키고 있다.
슈퍼타입의 참조변수가 이 힙영역의 서브타입의 인스턴스를 가리키고 있을 때 슈퍼타입을 서브타입의 참조변수로 형변환하고 이 서브타입의 참조변수로
같은 인스턴스를 가리키게 하면 단순히 인스턴스는 그대로이고 참조변수의 형만 변환되는거기 때문에 서브클래스만의 필드들을 접근할 수 있다.


```java

class PBoard { }

class CBoard extends PBoard { }

public class ClassCast {
    public static void main(String[] args) {
        PBoard sbd1 = new CBoard();
        CBoard sbd2 = (CBoard) sbd1;

        System.out.println("------------------------");

        PBoard ebd11 = new PBoard();
        CBoard ebd2 = (CBoard) ebd1;    // 이건 슈퍼클래스 인스턴스를 서브클래스의 참조변수로 가리키려고 하기때문에 오류.
    }
}
```


### 은닉화

객체의 변수를 public으로 설정하면 외부에서 마음대로 이 변수를 사용할 수 있다.
- 의도하지 않은 범위의 값을 넣을 수 있다.
원하지 않는 데이터타입을 강제적으로 형변환하여 넣을 수도 있다.

원하지 않는 데이터 넣는것을 방지하기 위해 Java Beans 패턴 (Getter/Setter) 를 사용.

이게 은닉화였네 몰랐슈

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


### 객체 확인

instanceof는 오브젝트가 지정한 클래스의 오브젝트인지를 조사하기 위한 연산자이다.
왼쪽 오브젝트가 오른쪽 클래스 또는 서브 클래스의 오브젝트라면 true

instanceof 는 지정한 인터페이스를 오브젝트가 구현하고 있는지를 조사할 수도 있다.
왼쪽 오브젝트가 오른쪽 인터페이스를 구현하고 있으면 true\

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

자바의 모든 클래스와 인터페이스는 컴파일 후 class 파일로 생성됨
class 파일에는 객체의 정보(멤버변수, 메서드, 생성자 등)가 포함되어 있음
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


### 절차지향 및 객체지향

절차지향 프로그래밍 : C

객체지향 프로그래밍 : Java, C++, C#, Python

절차지향과 객체지향 개념은 반대되는 것이 아니다. 서로가 보완될수도 있는 것이다.

자동차를 만들기 위해서는 차체, 바퀴,엔진, 핸들 ,의자, 엑셀 등등 많은 부품이들이 있어야 한다.

절차지향 프로그래밍의 경우 집안 대대로 차체 -> 바퀴 -> 엔진 -> 핸들 순으로 작업 했다. 이 순서가 아니면 우리 집안의 제품이 아니다.

객체지향의 프로그래밍의 경우는 공장에서 각 공장에서 각 요소들의 제품들을 제작한다.
제작 순서 자체는 중요하지 않다. 물론 비동기 적으로 동작할수도 있고 아닐 수도 있다.

절차지향의 프로그래밍은 순차적인 처리가 중요시 되며 프로그램 전체가 유기적으로 연결되도록 만드는 프로그래밍 기법
-> 트랜잭션 처리 등은 이걸로 해야할것 같은데

이들은 서로 분리되면 안되고 함수 호출 순서가 틀려서도 안되면 하나가 고장나면 전체 기능이 마비된다.
절차지향은 데이터를 중심으로 함수를 구성한다 함수의 호출 순서가 바뀌면 데이터의 전달과 값이 변할 수 있따.

객체지향 프로그래밍은 개발하려는 것을 기능별ㄹ ㅗ묶어 모듈화를 하고 모듈을 재활용하는 프로그래밍 방식.

제작에 있어서 순서적이지 않아도 된다.
이들은 각각 따로 독립적으로 개발되어 나중에 한곳에 모여 자신의 기능만 제대로 발휘하면 된다.
부품들이 결합되어 움직이다 어느 하나가 고장이 나면 고장난 부품만 고쳐주면 되고 다른 부품들은 영향을 받지 않는다.
필요한 부품을 다른 것으로 교체할 수 있다.
