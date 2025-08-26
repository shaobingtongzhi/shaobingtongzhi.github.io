---
title: Springboot中集成Jasypt
date: 2025-08-26
categories: 
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - 加密
  - 配置加密
  - 密码加密
---

# 简介

Jasypt（Java Simplified Encryption）是一个非常实用的 Java 加密库，旨在简化加密操作，帮助开发人员轻松地在 Java 应用中集成加密功能。它的全名是 "Java Simplified Encryption"，表示它提供了一种简单的方式来对数据进行加密和解密。Spring Boot 集成 Jasypt 进行配置文件加密和解密，能有效地保护敏感信息如数据库密码、API 密钥等。

# 步骤

## 添加 Jasypt 依赖

在 `pom.xml`中添加`Jasypt`的依赖

```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.5</version>
</dependency>
```

## 配置加密算法

在`application.yml`中设置`Jasypt`的加密密钥

```yml
jasypt:
  encryptor:
    password: my-secret-password
```

> 这里 `my-secret-password` 是加密和解密所用的密码，实际项目中要将其保存在安全的地方，避免暴露

## 配置敏感数据加密

```yml
# 加密的核心库路径
core:
  native:
    library: "ENC(6ZyzAYoXNybKYaiEn6iifUxDk4n7nR4uL9Cr8sslYPw8taPtZVzT2Q==)"
```

加密格式为 `ENC(密文)`，其中密文是使用 Jasypt 加密工具加密得到的。

## 使用`Jasypt`生成加密密文

可以使用 Jasypt 提供的命令行工具来加密明文数据，获取加密后的密文。

- 下载 Jasypt Command Line Tool。
- 使用下面的命令来加密文本（例如数据库密码）：

```sh
java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="/native/MacValidator.dll" password="123456" algorithm="PBEWithMD5AndDES"
```

这将返回一个加密后的字符串，将其复制到配置文件中的 `ENC(...)` 里。

## 使用加密的配置

一旦将加密后的值放入配置文件中，Jasypt 会在 Spring Boot 启动时自动解密它们，解密后的值会被注入到相应的属性中。

例如，如果你的配置是：

```yml
spring:
	datasource:
		password: ENC(9cfcf3fd5adcbfb8375e0e1c3a801830)
```

Spring Boot 会自动解密这个字段，将 `spring.datasource.password` 的值注入到 `DataSource` 配置中。

## 测试配置

确保 Spring Boot 启动时，相关的加密配置已正确解密并生效。你可以通过打印相应的值来检查它是否正确解密。

在一个 Spring Bean 中，通过 `@Value` 注解注入 `core.native.library` 配置项并打印它的值：

```java
@SpringBootTest
public class TestApplication {
    @Value("${core.native.library}")
    private String library;
    @Test
    public void test(){
        System.out.println("library:"+library);
    }
}
```

## 扩展

通过自定义的bean实现对配置的管理

```java
import org.jasypt.encryption.StringEncryptor;
import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.apache.commons.lang3.StringUtils;

@Configuration
public class JasyptConfig {

    @Value("${jasypt.encryptor.password:123456}")  // 通过配置文件获取密码，默认密码为123456（应更改）
    private String password;

    @Value("${jasypt.encryptor.algorithm:PBEWithMD5AndDES}")  // 通过配置文件获取加密算法，默认使用PBEWithMD5AndDES
    private String algorithm;

    @Value("${jasypt.encryptor.poolSize:1}")  // 配置线程池大小，默认1
    private int poolSize;

    @Bean("jasyptStringEncryptor")
    public StringEncryptor stringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();

        // 获取密码（优先级：系统属性 > 环境变量 > 配置文件 > 默认值）
        String effectivePassword = getPassword();
        config.setPassword(effectivePassword);

        // 配置加密算法、线程池大小和输出格式
        config.setAlgorithm(algorithm);
        config.setPoolSize(poolSize);
        config.setStringOutputType("base64");  // 确保输出为文本格式

        encryptor.setConfig(config);
        return encryptor;
    }

    private String getPassword() {
        // 优先级：系统属性 > 环境变量 > 配置文件中注入的值 > 默认值
        String passwordFromSystemProp = System.getProperty("jasypt.encryptor.password");
        if (StringUtils.isNotBlank(passwordFromSystemProp)) {
            return passwordFromSystemProp;
        }
        
        String passwordFromEnv = System.getenv("JASYPT_ENCRYPTOR_PASSWORD");
        if (StringUtils.isNotBlank(passwordFromEnv)) {
            return passwordFromEnv;
        }

        // 如果没有找到密码，返回配置文件中的密码或者默认值
        return StringUtils.defaultIfBlank(password, "123456");  // 使用配置文件或默认值
    }
}


```

