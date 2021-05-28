---
layout: post
title:  "Basic Gradle"
date:   2021-05-28 00:0054 +0900
categories: gradle
---

# apply plugin: 'java'

apply plugin 키워드는 Gradle 플러그인을 사용하기 위함
'java' 는 Java 프로그램을 위한 기능을 제공하는 플러그인이다.

# buildscript

buildscript 안에서 정의된 dependencies는 task를 사용할 때 사용되고, 밖에서 정의된 dependencies는 소스를 컴파일할때 사용.

# compile vs implementation