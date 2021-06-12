---
layout: post
title:  "Serialize in Java"
date:   2021-06-11 00:0054 +0900
categories: java
---

## @SerializedName

Gson 라이브러리에서 객체를 json으로 serialize 할때 key 값을 바꿔주고 싶은 경우 해당 필드에 이 어노테이션을 적용 한다.

```java

@SerializedName("Header")
private ResHeader header;

```

## @Transient

serialize 될때 특정 필드에 이 어노테이션을 달아주면 그 필드는 생략된다.
