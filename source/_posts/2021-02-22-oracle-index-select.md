---
layout: post
title:  "Oracle index select"
date:   2021-02-22 00:0054 +0900
categories: java
---

```
SELECT A.TABLE_NAME
     , A.INDEX_NAME
     , A.COLUMN_NAME
  FROM ALL_IND_COLUMNS A
 WHERE A.TABLE_NAME = 'EMP'
 ORDER BY A.INDEX_NAME, A.COLUMN_POSITION
```