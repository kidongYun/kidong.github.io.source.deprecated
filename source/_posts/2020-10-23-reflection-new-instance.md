---
layout: post
title:  "Creating new instance using Reflection technique"
date:   2020-10-23 11:38:54 +0900
categories: java
---

Creating new instance using Reflection technique means You would call the contructor of the class. 
First of all, you gotta set the parameters for the constructor.

### Create Class<?> by Class name

```java
Class<T> clazz = (Class<T>) Class.forName("CLASSNAME");
```

You could do either like this.

### Setting Parameters

```java
Class[] classArgs = {Map.class, Map.class};
```

### Setting Contruction method 

```java
Constructor constructor = clazz.getDeclaredConstructor(classArgs);
```

### Create new instance

```java
Object obj = constructor.newInstance(map1, map2);
```