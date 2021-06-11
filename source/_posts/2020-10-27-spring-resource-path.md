---
layout: post
title:  "Spring resource path"
date:   2020-10-27 11:38:54 +0900
categories: spring
---

```java

    String path = new File(BatchLogWorkerThread.class.getResource("/").getPath()).getParent() + "/";
    return new File(path + new SimpleDateFormat("yyyyMMdd").format(date) + "_batchLog.xlsx");
    
```