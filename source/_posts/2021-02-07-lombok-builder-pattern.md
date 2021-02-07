---
layout: post
title:  "Lombok Builder Pattern"
date:   2021-02-07 00:0054 +0900
categories: java
---

### AllArgs 을 위한 builder는 만들지마라 -> 클래스단위로 Builder를 만들지 마라
그러면 모든 매개변수를 변경할 수 있는 기회를 가지게 된다. 접근제어가 필요한 매개변수들은 제어를 해주는게 좋다는 의미.
이를 해결하기 위해서는 클래스단위로 빌더를 선언하는 거보다는 메소드 별로 빌더를 선언하는 것이 좋다.

### build() 이후에 callback 처리하기.
아직 lombok builder 패턴에서는 이를 @postbuild 어노테이션 같은걸 지원하지 않는다.
그래서 callback 처리 하려면 커스텀하게 Builder 클래스를 구현해야한다.
어떤 원리인지 모르겠으나 동일한 클래스명으로 정의하면 해당 클래스가 상속되어서 구현된다.
아래같이 구현하면 된다.

```java

@Slf4j
@Getter
@ToString
public class Note {
    private final Key key;
    private final Integer octave;
    private Integer pitch;

    @Builder(buildMethodName = "buildInternal")
    private Note(Key key, Integer octave) {
        this.key = key;
        this.octave = octave;
    }

    public static class NoteBuilder {
        public Note build() {
            Note note = this.buildInternal();
            // 여기서 콜백 처리
            return note;
        }
    }
}

```

buildMethodName 옵션에 값을 적용하면 default로 build() 함수의 역할의 이름이 바뀐다. 기본적으로 제공하는 build() 함수의 이름을 buildInternal() 로 변경하고
새로 Builder 클래스를 만들었다.
그리고 먼저 buildInternal() 이 함수를 호출한다음에 그다음 콜백처리할 내용을 작업하고 Note 객체를 리턴한다.

### Static Factory Method 명명을 어떻게 해야할까.

