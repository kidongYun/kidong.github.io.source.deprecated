---
layout: post
title:  "Object mapping using Modelmapper and Mapstruct"
date:   2021-06-04 00:0054 +0900
categories: java
---

## ModelMapper

You should add the below dependency to use model mapper.

```xml
        <dependency>
            <groupId>org.modelmapper</groupId>
            <artifactId>modelmapper</artifactId>
            <version>2.3.0</version>
        </dependency>
```

and you should insert @Setter, @NoArgsConstructor, @AllArgscontstructor annotation when you make your dto file or domain. like below.

```java

@Getter
@Setter
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class BookDTO {
    private String title;
    private String author;
}

@Getter
@Setter
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Book {
    private String title;
    private String author;
}

```

then you can convert each other using map() method of Modelmapper instance.

```java
    @Test
    public void test_use_model_mapper() {
        ModelMapper modelMapper = new ModelMapper();

        BookDTO bookDTO = BookDTO.builder()
                .title("Selfish Gene").author("Lichard Dokins").build();

        Book book = modelMapper.map(bookDTO, Book.class);

        log.info("YKD : " + book.toString());
    }
```

Let's use instance type like Long or CurrencyType of which you could see the below code

```java

@Getter
@Setter
@Builder
@ToString
public class CurrencyType {
    private final String type;
}

@Getter
@Setter
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class BookDTO {
    private String title;
    private String author;
    private CurrencyType currencyType;
    private LocalDate publishedAt;
    private Long price;
}

@Getter
@Setter
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Book {
    private String title;
    private String author;
    private CurrencyType currencyType;
    private LocalDate createAt;
    private Long price;
}

```

it will work also good. the instance type field like CurrencyType is converted automatically without any setting from us. It's so cool.

```java

        ModelMapper modelMapper = new ModelMapper();

        BookDTO bookDTO = BookDTO.builder()
                .title("Selfish Gene")
                .author("Lichard Dokins")
                .price(24000L)
                .currencyType(CurrencyType.builder().type("KRW").build())
                .publishedAt(LocalDate.of(1984, 6, 20)).build();

        Book book = modelMapper.map(bookDTO, Book.class);

        log.info("YKD : " + book.toString());

```

but if you have some differences between the field name of DTO object and model object, then to convert each other you should map them.

```

```