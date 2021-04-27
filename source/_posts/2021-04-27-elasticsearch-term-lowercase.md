---
layout: post
title:  "Elasticsearch Term 쿼리 쓸때 소문자로 해야 함"
date:   2021-04-27 00:0054 +0900
categories: java
---

Elasticsearch Term 쿼리 쓸때 소문자로 해야 합니다.. 삽질 엄청 했습니다.

```json
{
    "size" : 100,
    "query" : {
        "terms" : {
            "airlineCode" : ["bx", "rs"]
        }
    }
}
```