---
layout: post
title:  "Spring Environment"
date:   2021-05-28 00:0054 +0900
categories: java
---

# Spring Environment

스프링의 환경이자 설정과 관련된 인터페이스.
Proiles 설정과 Property 설정에 접근할 수 있다.
ApplicationContext 안에 해당 객체가 포함되어 있어서 ApplicationContext 로도 접근이 가능하다.

Profiles
스프링 웹 애플리케이션을 개발할 때, 애플리케이션 환경에 따라 약간씩 변경이 되어야할 설정 값들을 코드의 변경 없이 설정만으로 편리하게 변경할 수 있도록 제공하는 인터페이스이자 분리의 기준.

application.properties
application-local.properties
application-test.properties
위와 같이 설정 파일을 만들고. 이 파일에 각 환경에 맞는 값들을 넣는다.

VM-option 이나 다양한 방법으로 어떤 properties 파일을 접근할지를 어플리케이션 구동시에 선택할 수 있다.
@Profile("test") 어노테이션으로도 접근 가능.

# UtilityClass

@UtilityClass
유틸리티 클래스에 적용하면 되는 어노테이션이다. 만약 이 어노테이션을 작성하면 기본생성자가 private 생성되며 만약 리플렉션 혹은 내부에서 생성자를 호출할 경우에는 UnsupportedOperationException이 발생한다.

## acceptsProfiles(String... profiles)

실행되는 프로파일이 profiles 파라미터에 있는 프로파일이라면 true 를 반환하고 그렇지 않으면 false 를 반환한다.

