---
layout: post
title:  "Intellij .http client"
date:   2021-05-31 00:0054 +0900
categories: java
---

## .http 장점

1. Git을 활용한 형상관리가 가능하다.
    Postman의 경우 소스코드와는 별개로 관리되어야하며 그렇기 때문에 협업 개발자들과 최신버전으로 공유하는 것도 쉽지 않다.
    Swagger의 경우 소스코드와 함께 적용하기 때문에 Git 과 같은 형상관리 프로그램에 적용이 될 수 있지만. 프로덕션 실제 소스에 함께 적용되기 때문에 분리가 되지 않는다.
    그리고 애초에 API call을 위해 단순히 테스트 해보는 용도라기 보다 Api Docs를 만들기 위한 것에 가깝다.

2. Intellij IDE 안에서 바로 Api Call 테스트가 가능하다.
    굳이 Postman 과 같이 다른 프로그램을 동작시킬 필요없이 Intellij 안에서 테스트해볼 수 있어서 화면전환이 필요없다.
    생산성이 충분히 늘어날 수 잇는 부분이다.


## 예시

```

### GET request with a header
GET https://httpbin.org/ip
Accept: application/json

### GET request with parameter
GET https://httpbin.org/get?show_env=1
Accept: application/json

### GET request with environment variables
GET {{host}}/get?show_env={{show_env}}
Accept: application/json

### GET request with disabled redirects
# @no-redirect
GET http://httpbin.org/status/301

### GET request with dynamic variables
GET http://httpbin.org/anything?id={{$uuid}}&ts={{$timestamp}}

```

HTTP Method가 POST 라면 아래처럼 Query String, Request body 를 전송할 수 있다.

```

### Get Token
POST http://localhost/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&scope=read_profile

git

> {% //response handler
client.global.set("access_token", response.body.access_token);
client.log(client.global.get("access_token"));
%}

```