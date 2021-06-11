---
layout: post
title:  "Jvm options"
date:   2020-09-03 10:00:00 +0900
categories: java
---


## SET

### Jvm argument
Jvm arguments are begun with '-D'.

```
java -jar -Dfoo heumsi-springboot.jar
```

### Application argument
Application arguments are begun with '--'.

```
java -jar --bar heumsi-sprinboot.jar
```

## GET

### application.properties

the arguments can be stored at 'application.properties' file as key-value type. and It can is approached using @Value in Java code.

```json
// set value in application.properties
heumsi.name = heumsi

// get value in *.java
@Value("${heumsi.name}")
private String name;
```

### Environment

You can approach the arguments without '.properties' file using Environment object.

```java
@Autowired
Environment environment;

private String name = environment.getProperty("heumsi.name"); 
```