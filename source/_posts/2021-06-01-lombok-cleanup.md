---
layout: post
title:  "Cleanup annotation of Lombok"
date:   2021-06-01 00:0054 +0900
categories: java
---

It's for calling 'close()' medthods safely with no hassle.

리소스를 사용하고 난 다음에 자동으로 사용되고있는 자원들을 정리해준다.

try-resource 문법과 같은 목적.

기존 vanila java 소스
```java

import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}

```

Cleanup 어노테이션 사용했을 때의 소스

```java

import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}

```