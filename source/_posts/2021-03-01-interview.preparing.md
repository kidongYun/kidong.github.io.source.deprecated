---
layout: post
title:  "Interview Preparing"
date:   2021-03-01 00:0054 +0900
categories: java
---

## 1. 데드락 (교착상태)

Mutual Exclusion (상호배제)

자원은 한번에 한 프로세스만이 사용할 수 있따.


## 2. Transaction

DBMS는 기본적으로 질의 처리기와 저장 시스템 두 부분으로 나누어진다.

그리고 저장 시스템에 메인 메모리에 유지하는 페이지들을 관리하는 페이지 버퍼라는 모듈이있다.
이 페이지 버퍼라는 모듈이 트랜잭션 과리를 하는데 매우 중요한 결정을 한다.

## 3. IoC (Inversion Of Control)

의존성 주입의 책임을. 개발자에게 넘기는게 아니고. 프레임워크에서 해준다. 제어의 역전이라고 해서 제어의 주체가 개발자가 아닌 프레임워크가 가져갔다는 의미.

의존성 관리라는 것이 커지면 커질수록 꽤나 부담이 되는 요소.

그렇기에 스프링 같은경우 스프링 컨테이너에서 빈들을 등록해두고. 여기서 의존성들을 주입시켜줄수 있또록 한다.

## 4. MVC (Model View Controller)

Model => 데이터를 가공하고 관리하는 영역
Persistant Layer 같은 곳.

View => 타임리프, jsp 같은 뷰 사이드.
여기서 보여지는 데이터들은 이 뷰영역에서 만들어지는 것이 아니고 Controller에서 받는다.

Controller -> 중간에서 Model 과 View 사이에서 매핑해준다.

장점 : MSA 중 하나의 패턴이라고 생각한다. 모듈화가 되기 때문에 어플리케이션의 유연성, 확장성이 증가된다.

## 5. 멀티쓰레드 vs 이벤트 루프 (nginx, apache)

### Prefork MPM
MPM 방식 (Multi Process Module) - prefork
MPM-prework : 멀티 프로세스 방식 마스터 프로세스가 있고 요청이 들어오면 슬레이브 프로세스를 fork 하여 만들고 이 슬레이브 프로세스에서 요청에 대한 처리를 한다.
매 요청에 맞춰서 슬레이브 프로세스를 만들어야 하기때문에 프로세스가 계속 늘어나는 문제점과, 매번 요청에 따라서 슬레이브 프로세스로 바꾸는 context switching 비용이 있다.

### Worker MPM
MPM-worker : 멀티 쓰레드 방식. 위 방식과 거의 유사하며 프로세스 대신 쓰레드를 사용하는 것이 다르다.

풀을 쓴다고 해도 쓰레드, 프로세스 수를 줄이는 것에 대한 근본적인 해결책이 될 수 없음.

apache 는 기본적으로 REQ가 들어오면 그거에 맞춰서 새로운 Process, Thread를 생성한다.

event-driven
reactor pattern
