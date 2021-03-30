---
layout: post
title:  "2차 면접 준비"
date:   2021-03-23 00:0054 +0900
categories: java
---

### 1. TCP

1. 특징
    - 제어를 한다.
    - 가상회선을 구축하고 해당 회선을 따라서 데이터를 주고 받는다.
    - 주고받는 데이터가 오류가 있으면 안될때 사용.

2. TCP Header
    - 32비트
    - SYN, FIN, ACK 플래그 존재.
    - SYN : 연결을 맺고싶을 때
    - FIN : 연결을 끊고싶을 때
    - ACK : 상대방의 정보를 잘 받았을 때.

3. 3 ways HandShake
    - 클라이언트가 SYN 신호가 1이 된 상태로 전송
    - 서버가 이를 수신했다는 의미로 ACK 신호를 1로 바꾸고, 마찬가지로 SYN 신호도 1로 바꾸어 전송
    - 클라이언트가 서버의 데이터를 수신했다는 의미로 ACK 플래그를 1로 바꾸어 전송.

4. 4 ways handshake
    - 클라이언트가 FIN 플래그를 전송
    - 서버가 수신완료를 의미하기 위해 ACK 플래그를 전송
    - 서버가 작업이 모두 완료되면 FIN 플래그를 전송
    - 클라이언트가 수신완료를 알리기 위해 ACK 플래그를 전송

### 2. TLS
    Transport Layer Security
    netscape사에서 처음으로 발표한 SSL 기술을 IETF 에서 표준적으로 정의한 용어.

1. 배경
    - 기존 HTTP 데이터를 암호화하지 않고 주고 받는 다면 DCE 장비들에서 데이터들이 노출이 됨.
    - 잘못된 상대방과 통신할 가능성이 있음.

2. 기능
    - 암복호화
    - 인증
    - 무결성

### 3. 웹 통신 흐름

- 웹브라우저 특정 URL 접속.
- DNS 서버 탐색을 해서 IP를 얻어냄 (local hosts 파일, 로컬 DNS 서버)
- ARP 프로토콜 IP -> MAC 주소로 변경.
- MAC 주소에 해당하는 Destination terminal을 찾아간다 (OSPF, RIP)
- 포트번호를 통해서 웹서버 프로세스에 접근한다.
- 정적 리소스 파일 요청이라면 웹서버에서 처리.
- 동적 컨텐츠라면 WAS에게 넘긴다.
- WAS에 있는 서블릿 컨테이너가 DispatcherServlet 을 생성.
- DispatcherServlet이 RequestMapping에 맞춰서 실행.
- view 파일이라면 resolver 에 맞춰서 응답 값 전달. 아니면 String.

### 4. 웹서버와 WAS를 구분시켜서 두는 이유.

1. 로드밸런싱 (modjk)
2. 보안 (WAS는 데이터베이스 등을 접근하기 때문에 외부로부터 노출 X)
3. 캐시 적중률 향상, 페이지 부재 감소

### 5. Sync vs Async

시간이 주요 관점.

### 6. Blocking vs Non Blocking

제어가 주요 관점.

### 7. 함수형 프로그래밍.

특징 
1. 멱등성 : 동일한 입력에 대해서 동일한 출력이 나온다.
2. 순수함수 : 멱등성을 지키는 함수. (의존성 문제, 테스트 코드 작성)
3. 일급객체 : 함수를 일급객체로 볼 수 있다.
4. 선언적 프로그래밍 방식 : 어떻게 할까보다 무엇을 할까가 포커스.

자바에서의 함수형 프로그래밍
1. 익명클래스
2. 람다, 메서드 레퍼런스.
3. 스트림.

### 8. 프로세스

1. 개념 : 메모리에 상주되어 있는 프로그램.

2. PCB 
        - 메모리 영역 (TEXT, DATA, HEAP, STACK)
        - PID : 프로세스 식별자.
        - 프로세스 상태 : new, ready, running, waiting, halted
        - 프로그램 카운터 : 다음 실행할 명령어의 주소.

3. IPC

### 9. Docker 

### 10. MVC

1. Model, View, Controller의 역할
2. 업무의 분할, 디버깅이 용이

MODEL-VIEW-CONTROLLER

### 11. elasticsearch

1. 노드, 샤드, 레플리카
2. 역색인
3. querydsl
4. 구조
    - 키바나 : 로그 모니터링
    - 엘라스틱서치 : 데이터 저장 및 데이터 조회
    - 로그스태쉬 : 다른 저장소와 연동

### 12. GC

1. Stop the worlld
2. eden, survivor1, survivor2, old, minorGC, fullGC
3. Parallel GC, CMS GC, G1GC


### 13. JVM

1. Class Loader - 디스크에 존재하는 class파일을 메인메모리에 적재.
2. Runtime data area - Heap, Method, Stack, 
3. GC
4. Execution engine - 자바 인터프리터, JIT 컴파일러.

### 14. Spring Framework.

1. Spring IOC, DL, DI, AOP
2. Spring Web MVC, Spring Data JPA, Spring DAta redis, Spring test

### 15. Spring AOP

1. Advice - 언제.
2. Weaving - Aspect를 Target에 넣는 행위 자체.
3. JointPoint - Advice가 적용 가능한 지점.
4. Aspect - 공통 기능.
5. Target - 공통 기능을 넣을 대상 객체


### 16. 캐시

1. 파레토 법칙 -> 개념
2. 지역성
3. Look aside, Write back

### 17. SOLID

1. S : 단일책임의 원칙.
- 하나의 클래스는 하나의 기능을 가진다.

2. O : 개방폐쇄의 원칙
- 코드를 확장하는 것에는 열려있고, 변경에는 닫혀있어야하는 원리

3. L : 리스코브 치환의 원칙.
- 서브타입은 언제나 기반타입과 호환이 될 수 있어야 한다.

4. I : 인터페이스 분리의 원칙.
- 하나의 일반적인 인터페이스 보다는 여러 개의 구체적인 인터페이스가 좋다.

5. D : 의존성 역전의 원칙.
- 

### 18. TDD

테스트 코드를 먼저 작성하고 소스 코드를 작성하는 방법.

단점 : TDD를 하면 개발 시간이 늘어난다.

장점 : 
    TDD를 하면 결함이 줄어든다. 지속적으로 테스트를 하면서 코드를 작성하기 때문.
    TDD를 하면 소스코드의 의존성을 감소시킬 수 있다.
    리팩토링에 대한 비용이 줄어든다.

### 19. FunctionalInterface

Consumer<T> -> 입력 : T타입, 출력 : void
Supplicer<T> -> 입력 : void, 출력 : T타입
Function<T, R> -> 입력 : T, 출력 : R
Predicate<T> -> 입력 : T, 출력 : boolean

### 20. Hash

1. 해시 기본 구조 (해시코드, 해시테이블)
2. 해시충돌 (체이닝, 이중해시)

### 21. Heap

트리구조, 자식노드가 루트노드보다 우선순위가 높지 않다.
삽입, 삭제 동작원리.

### 22. 이진 탐색 트리

### 23. JPA (ORM의 장점)

벤더사 독립적이다.

persistent context 라는 캐싱 영역이 존재한다.

트랜잭션 관리가 용이하다.

CRUD 쿼리를 직접 짜지않기 때문에 생산성이 좋아진다.

### 24. N+1 문제

이렇게 하위 엔티티들을 첫 쿼리 실행시 한번에 가져오지 않고, Lazy Loading으로 필요한 곳에서 사용되어 쿼리가 실행될때 발생하는 문제가 N+1 쿼리 문제입니다.

해결방법 : joinFetch, EntityGraph

### 25. 인덱스.

1. B-TREE 구조.
2. INDEX RANGE SCAN, FULL TABLE SCAN
3. INDEX ACCCESS 조건, FILTER 조건


### 26. JOIN

1. NETSTED LOOP JOIN
2. HASH JOIN
3. SORT MERGE JOIN

### 27. Checked Exception, Unchecked Exception

### 28. Redis

1. 메모리에서 동작.
    - 물리적 메모리 크기보다 크면, 디스크를 활용하게 되기 때문에 성능 감소 -> 레디스를 쓰는 이유가 사라짐.
2. Collection
    - Set, SortedSet, List, Hash,
3. RedisTemplate
4. 레디스는 싱글스레드 처리

### 29. 프로세스 vs 쓰레드

프로세스 : 프로그램이 메인 메모리에 올라감.
PCB : PID, 메모리(Heap, Text, Stack, Data), PC

쓰레드 : 프로세스 내부에서의 코드의 흐름.
Heap 영역 제외 공유.
데이터 공유 비용이 더 쌈.

IPC - Inter Process Communication

### 30. 쿠키, 세션, JWT

### 31. Spring security

1. Filter based 처리
2. 인증, 권한
3. Security Context


### 32. JWT

header.payload.signature

장점. 사용자 인증에 필요한 모든 정보는 토큰 자체에 포함하기 때문에 저장소 X, 확장 유지보수에서 유리

단점. 유효기간. -> Refresh Token 해결

### 33. REST

HTTP URL, HTTP Method

Open API

### 34. ACID

트랜잭션이란 논리적 작업 단위.

A : 원자성 - 해당 작업이 논리적으로 데이터 정합성 관점에서 더 이상 나눠질 수 없음.

C : 일관성 - 해당 트랜잭션의 결과가 항상 동일하다.

I : 독립성 - 해당 트랜잭션이 동작될 떄 다른 트랜잭션 개입 X

D : 지속성 - 해다 트랜잭션의 결과가 일시적이지 않고 영구히 지속되어야함.

### 35. 테스트 종류

유닛테스트

통합테스트

기능테스트


### 36. DP, Greedy, 

### 37. 빌더패턴, 프록시패턴

### 38. nginx 로드밸런싱 적용.

1. upstream 설정
    분배규칙
    weight



### 39. Hash

hashAlgorithm을 설계 -> hashAlgorithm은 HashCode를 반환 -> hashCode를 hashtable 크기에 맞게 모듈러 연산 -> 이 결과 값이 배열의 키가 되며 해당 인덱스에 value 값을 저장

value 가 여러개일 경우 충돌이 발생했다 하며 해결 방법은 연결리스트를 두는 Chanining 방법, Open Addressing 방법이 있음.
여기서는 Chaining 방법을 고려

값이 여러개가 되면 연결리스트를 두고 차곡차곡 쌓음.
충돌이 많이 나는 경우에는 이 부분을 트리구조로 변경하여 검색의 속도를 높일 수 있음.

