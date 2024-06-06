---
title: Springboot代码中判断是否生产环境
date: 2024-06-04
categories:
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - Springboot
  - 配置
---



s

思路：

把系统的环境变量注入

工具类源码：

```java
@Component
public class MyComponent {
    private static Environment environment;

    @Autowired
    public MyComponent(Environment environment) {
        MyComponent.environment = environment;
    }

    public static String getConfigValue(String key) {
        return environment.getProperty(key);
    }
}
```

使用：

```java
String env = MyComponent.getConfigValue("spring.profiles.active");
```

