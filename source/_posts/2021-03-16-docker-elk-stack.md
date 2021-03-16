---
layout: post
title:  "도커기반 ELK Stack 구축"
date:   2021-03-16 00:0054 +0900
categories: java
---

1. 도커 활용하여 elasticsearch 설치

```
docker run -d -p 9200:9200 -p 9300:9300 -it -h elasticsearch --name elasticsearch elasticsearch:7.9.3
```

버전 명시를 안하니까 docker hub 에서 이미지가 pull 이 안되었다.

```
docker search elasticsearch
```

위 키워드로 공식 이미지 명을 찾은 다음.

```
https://registry.hub.docker.com/v1/repositories/elasticsearch/tags
```
위 주소 접속하여 가장 최신 버전의 태그를 pull 받았다.

위 주소에서 elasticsearch 부분만 원하는 이미지 명으로 변경하면 다른 이미지들도 태그 확인이 가능하다.

--------------

elk 를 다 구축해둔 깃 저장소가 있음.
이거를 클론해서 설정만 좀 변경하고.
docker-compose 로 실행해보겠음.

https://judo0179.tistory.com/60

요기 나온대로 따라서 소스 수정하고

docker-compose 로 실행.

그러면 해당하는 이미지들을 pull 받고 (FROM 키워드)
docker-compose 설정처럼 각 이미지들을 활용해 컨테이너를 띄운다.
