---
title: JAVA 基础
date: 2025-09-17
categories:
  - 
tags:
  - 
---

# 时间相关

## 转时间戳

```java
// 获取最近30天的数据
LocalDateTime endTime = LocalDateTime.now();
LocalDateTime startTime = endTime.minusDays(30);

// 转换为Unix时间戳
Integer startTimestamp = (int) startTime.toEpochSecond(java.time.ZoneOffset.ofHours(8));
Integer endTimestamp = (int) endTime.toEpochSecond(java.time.ZoneOffset.ofHours(8));
```

