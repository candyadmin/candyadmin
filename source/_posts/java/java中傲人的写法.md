---
title: java中傲人的写法
date: 2025-11-29 17:21:06
categories: Java
tag: 傲人的写法
---

```java
@ConditionalOnExpression("#{ T(org.springframework.boot.system.SystemProperties).get('os.name').toLowerCase().contains('linux') }")
```
