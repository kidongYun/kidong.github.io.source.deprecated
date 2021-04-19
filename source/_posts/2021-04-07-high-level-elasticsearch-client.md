---
layout: post
title:  "스프링 부트에 high level 엘라스틱서치 클라이언트 적용"
date:   2021-04-07 00:0054 +0900
categories: java
---

### dependencies

```xml
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
        </dependency>
```

### Config

```java

@Configuration
@AllArgsConstructor
public class ElasticsearchConfig {
    private final String host = "XXX.XXX.XXX.XXX";
    private final int port = 9200;

    @Bean
    public RestHighLevelClient client() {
        return new RestHighLevelClient(RestClient.builder(new HttpHost(host, port, "http")));
    }
}

```