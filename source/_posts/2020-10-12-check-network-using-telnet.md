---
layout: post
title:  "Checking network status using telnet command"
date:   2020-10-12 00:00:00 +0900
categories: network
---

telnet actually is operated on 23 port number. but if you just check the network status whether it's opened or blocked, you can use telnet command.

```
    > telnet 180.70.98.168 6393
```

If it is opened, It would give the messages like the below.

```
    Trying 180.70.98.168...
    Connected to 180.70.98.168.
    Escape character is '^]'.
```

Or not

```
    Trying 180.70.98.168...
```