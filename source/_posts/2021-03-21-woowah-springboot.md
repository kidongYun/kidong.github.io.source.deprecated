---
layout: post
title:  "우아한 스프링 부트"
date:   2021-03-21 00:0054 +0900
categories: java
---

packaging - jar, war
war 는 자바 웹어플리케이션 구조로 구축된 형태. WEB-INF 디렉토리를 가지고 있고 등등..

jar - java archive

war - web application archive

jar, war 차이점 찾아보기.

spring boot 가 제공하는 첫번째 기능
-> 프로젝트를 생성하기가 간편하다. dispatcher-servlet 설정 등을 자동설정 해준다.

두번째 기능
-> 버전관리.
빌드툴(메이븐, 그래들) 에서 특정 프로젝트를 가져 올 때 필요한 정보는 groupId, artifactId, version 인데 스프링부트에서 의존성 관리부분을 보면 버전 정보가 없는 것들이 있다. 이거는 메이븐의 경우 parent 에서 의존성을 미리 관리해주는 의존성들이 있는데 이것들은 버전들이 이미 관리되고 있기 떄문에 버전을 명시하지 않아도 된다.

가장 호환이 잘되는 버전을 찾는것 자체가 일이였음. 그걸 스프링 부트에서 관리해주기 때문에 하지 않아도 된다.

mvnrepository.com - maven 중앙저장소에 있는 의존성들을 검색하고 다운받을 수 있는 사이트.

스프링부트 프로젝트를 띄우는 방법.

1. ide를 활용해서 main() 함수 실행하기.

2.  mvn spring-boot:run 명령어 실행.

#### ./mvnw 파일은??
./mvnw 이 파일이 mvn 이 설치되어있지 않아도 mvn 을 사용할 수 있도록 하는 스크립크가 작성되어 있음.

3. 서버에서 어플리케이션을 패키징한 후 해당 파일을 실행시키는 방법.(mvn package)

mvn package 하면 /target 디렉토리에 jar 파일로 하나로 뭉쳐진다.

의존성, 라이브러리, 소스코드를 모두 포함해서 jar 파일을 패키징 한다.

기본적으로 이렇게 패키징할 때 자바에서 제안하는 표준은 없고 스프링 부트가 자체적으로 만든것.

