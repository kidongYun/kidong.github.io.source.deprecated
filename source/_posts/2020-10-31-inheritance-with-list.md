---
layout: post
title:  "The problem of mismatching type when you use List."
date:   2020-10-27 11:38:54 +0900
categories: spring
---

Let's think just a really simple struture for understanding of inheritance concept.

```java

class A {

}

class B extends A {

}

class C {

}

```

Class A is a super class and Class B is extended Class A. Class C is independent thing. It doesn't have any relationship with other classes.

```java

    /** It's possible */
    A a = new A();

    /** It's possible because B is sub-class of A. */
    A a = new B();

    /** It's impossible because C doesn't have any relationship with A. */
    A a = new C();

```

You could understand the above code. but let's see the below, which i haven't understood well just before now.

```java

    /** It's possible. */
    ArrayList<A> aList = new ArrayList<A>;

    /** It's possible. */
    aList.add(new B());

    /** It's impossible. */
    aList = new ArrayList<B>;

```

The last one is the main topic of this article. I usually think that code would work well. but it's not. the reason why it isn't work well is you should understand ArrayList<B> is not extended ArrayList<A>