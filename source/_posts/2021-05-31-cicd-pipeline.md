---
layout: post
title:  "CICD Pipeline"
date:   2021-05-31 00:0054 +0900
categories: java
---

## Jenkins (Continuous Integration Tool)
소프트웨어 개발 시에 지속적 통합 서비스를 제공하는 오픈소스 툴. 보통 원격 리포지토리에 jenkins의 webhook을 걸어서 특정 브랜치가 push 될 때 jenkins가 그 원격 리포지토리의 소스코드를 바탕으로 미리 정의해놓은 빌드, 테스트, 배포의 절차를 실행하는 방식으로 동작합니다.

Jenkins를 통해서 새로운 코드를 테스트하고, 빌드한 후 빌드 파일을 Docker image로 만들어서 쿠버네티스 시스템에서 새로운 코드가 돌아갈 수 잇도록 파이프라인 구성.

