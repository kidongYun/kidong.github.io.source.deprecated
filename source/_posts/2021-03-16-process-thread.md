---
layout: post
title:  "Process vs Thread"
date:   2021-03-16 00:0054 +0900
categories: java
---

Program.
명령어, 코드 및 정적인 데이터의 묶음

Process
실행 중인 Program
운영체제로부터 시스템 자원을 할당 받는 작업의 단위.

프로세스의 4개의 메모리 영역
stack -> 매개변수, 지역변수
heap -> 동적으로 할당되는 메모리
data -> 정적 변수, 상수 (jvm에서 method 영역)
text -> 프로그램의 코드

운영체제가 Process를 관리하기 위해 PCB 라는 것을 정해뒀음.

PCB (Process Control Block)
각 프로세스는 운영체제에서 PCB로 표현.
PID : 프로세스 식별자.
프로세스 상태 : new, ready, running, waiting, halted
프로그램 카운터 : 다음 실행할 명령어의 주소.
스케줄리 정보 : 우선순위 등.

쓰레드
프로세스 내에서 실행되는 흐름의 단위.
CPU 이용의 기본 단위.

쓰레드는 한 프로세스 내에서 여러개 생성 될 수 있으며
한 프로세스의 Text, Data, Heap 영역의 자료를 공유한다.
그러나 각 Thread 별로 stack 영역은 별도로 가진다.

Multi Thread vs Multi Process

Multi Thread
프로세스의 자원을 공유
향상된 응답성
Context Switching 비용이 적음.
자원을 공유하는 만큼 충돌을 주의 (Thread-safe하게) -> 임계영역이 됨.

Multi Process
하나의 작업을 여러 개의 프로세스가 처리
프로세스간 통신 (IPC : Interprocess communication)
Context switching 비용이 큼
자식 프로세스 중 하나가 문제가 생겨도 다른 프로세스에 영향이 없음.
