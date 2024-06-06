---
title: Springboot读取自定义的yml配置文件
date: 2024-06-04
categories:
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - 自定义yml
---





在开发中经常会遇到读取自定义的yml配置文件

第一步：添加配置类

```java
@Configuration
public class LoadCustomYmlConfig {
    @Bean
    public YamlPropertiesFactoryBean alarmParamBean(){
        YamlPropertiesFactoryBean ypfb = new YamlPropertiesFactoryBean();
        ypfb.setResources(new ClassPathResource("custom/alarmParam.yml"));
        return ypfb;
    }
}
```

第二步：添加实体类，用来获取配置

```java
@Component
public class AlarmParamYml {
    @Resource(name = "alarmParamBean")
    private Properties alarmParam;
    public Properties getAlarmParam() {
        return alarmParam;
    }
}
```

第三步：注入实体，使用

```java
@Autowired
private AlarmParamYml alarmParamYml;

....
alarmParamYml.getAlarmParam().getProperty("配置项")
....
```



@Resource 和 @Autowired 

在java代码中使用@Autowired或@Resource注解方式进行装配，这两个注解的区别是：@Autowired 默认按类型装配，@Resource默认按名称装配，当找不到与名称匹配的bean才会按类型装配
