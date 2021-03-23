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

MODEL-VIEW-CONTROLLER








