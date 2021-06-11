---
layout: post
title:  "함수형 인터페이스 정리해두기"
date:   2020-12-29 20:00:00 +0900
categories: java
---

```java
@FunctionalInterface
Consumer<T> {
    void accept(T t)
}	
```

```java
@FunctionalInterface
Supplier<T> {
    T get()
}
```

```java
@FunctionalInterface
Function<T, R>	{
    R apply(T t)
}
```	

```java
@FunctionalInterface
Predicate<T> {	
    boolean test(T t)
}
```
