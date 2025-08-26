---
title: Ry中common模块向core模块的演进
date: 2025-08-25
categories:
  - 杂文
  - 推荐
tags:
  - 核心模块
  - 加密授权
---

# 简介

原生项目是基于RuoYi的前后分离版进行的开发，开发过程中提出新的需求，需要对项目中的部分代码（核心代码）进行保护，根据这个需求诞生了下面的方案。这个core模块是将原本代码中的common模块修改而来。

# 核心模块方案

## 1. 概述

xxxx-core 是一个独立的核心模块，旨在提供系统级的核心功能，包括但不限于：

- 环境验证（通过本地库文件 .so/.dll）
- 核心业务逻辑保护
- 系统安全验证机制

该模块设计为完全独立，不依赖于主项目的构建流程，可以单独开发、构建和部署。

## 2. 设计目标

### 2.1 代码保护

- 将关键业务逻辑封装在本地库文件中（.so/.dll）
- 通过JNI接口提供Java层面的调用
- 保护核心算法和敏感信息

### 2.2 环境验证

- 在系统启动时验证运行环境
- 检查系统配置、文件完整性等
- 如果验证失败，阻止系统正常启动

### 2.3 独立部署

- 不集成到主项目构建流程中
- 可以独立版本控制和发布
- 通过Maven本地仓库机制供其他模块使用

## 3. 技术架构

### 3.1 模块依赖关系

```
xxxx-core 
└── 提供: 核心功能接口给其他模块使用
其他iescp-* 模块 
└── 依赖: xxxx-core (通过Maven依赖)
```

### 3.2 核心组件

#### 3.2.1 本地库加载器

负责加载和验证本地库文件（.so/.dll）：

- 检查库文件是否存在
- 验证库文件完整性
- 加载库文件到JVM

#### 3.2.2 环境验证器

在应用启动时执行环境验证：

- 系统信息验证
- 配置文件验证
- 授权信息验证

#### 3.2.3 核心业务逻辑

通过JNI调用本地库实现的关键业务逻辑。

## 4. 使用方案

### 4.1 构建xxxx-core模块

```bash
进入xxxx-core目录
cd xxxx-core
清理并安装到本地Maven仓库
mvn clean install
```

### 4.2 在其他模块中使用

在需要使用核心功能的模块pom.xml中添加依赖：

```xml
<dependency>
    <groupId>net.zhiboredian</groupId>
    <artifactId>xxxx-core</artifactId>
    <version>1.0.0</version> 
</dependency>
```

### 4.3 应用启动流程

1. 应用启动时自动触发xxxx-core的环境验证
2. 调用本地库文件进行环境检查
3. 如果验证失败，抛出异常阻止应用启动
4. 验证通过后，正常启动应用

## 5. 开发指南

### 5.1 目录结构

```
xxxx-core/ 
├── pom.xml 
├── src/ 
│ ├── main/ 
│ │ ├── java/ 
│ │ │ └── net/ 
│ │ │ └── zhiboredian/ 
│ │ │ └── core/ 
│ │ │ ├── CoreAutoConfiguration.java 
│ │ │ ├── validator/ 
│ │ │ │ └── EnvironmentValidator.java 
│ │ │ └── jni/ 
│ │ │ └── NativeLibraryLoader.java 
│ │ └── resources/ 
│ │ │       └─——— native
│ │ │         │──——encryption.c 
│ │ │         │─——─encryption.exe
│ │ │         │─——─MacValidator.c
│ │ │         │─———MacValidator.dll
│ │ │         └─———MacValidator.h
│ │ └──── application-verify.yml 
│ └── test/ 
│ └── java/ 
└── 核心模块说明.md

```

### 5.2 核心类说明

#### EnvironmentValidator

环境验证器，在Spring启动时自动执行验证逻辑。

#### NativeLibraryLoader

本地库加载器，负责加载和验证.so/.dll文件。

#### CoreAutoConfiguration

自动配置类，用于Spring Boot自动配置核心功能。

## 6. 部署方案

### 6.1 本地库文件部署

本地库文件（.so/.dll）需要部署在特定位置：

- Windows: `%APP_HOME%/native/library.dll`
- Linux: `$APP_HOME/native/library.so`

### 6.2 配置文件

核心模块的相关配置应放在`application-verify.yml`中：

```yml
core: 
  native: 
    library: ENC(BWO4AXdMkxE4xp4kpHXPwpD+k8hnJu1WbIrAG0U9H5ddg8nhTXxvZw==) #本地库文件路径,已加密(/native/MacValidator.dll)
  environment: 
    validation: 38dd3706256f1d66f90165149495e589 #环境验证开关(通过encryption.exe对true|false进行加密)
```

## 7. 版本管理

### 7.1 版本策略

采用语义化版本控制：

- 主版本号：不兼容的重大变更
- 次版本号：向后兼容的功能性新增
- 修订号：向后兼容的问题修正

### 7.2 发布流程

1. 更新版本号
2. 构建并安装到本地仓库
3. 在需要使用的模块中更新版本依赖

## 8. 安全考虑

### 8.1 本地库保护

- 本地库文件应进行数字签名
- 验证库文件的哈希值
- 防止库文件被篡改

### 8.2 环境验证

- 验证操作系统信息
- 验证硬件指纹
- 验证授权许可文件

## 9. 性能影响

### 9.1 启动时间

环境验证会增加应用启动时间，预计增加100-500ms。

### 9.2 内存占用

核心模块会增加约2-5MB的内存占用。

## 10. 故障处理

### 10.1 验证失败

当环境验证失败时，应：

- 记录详细的错误日志
- 提供清晰的错误信息
- 阻止应用继续启动

### 10.2 本地库加载失败

当本地库加载失败时，应：

- 检查库文件是否存在
- 验证系统架构兼容性
- 提供手动安装指导

# encryption 的开发与打包

简介：用来给`application-verify.yml`中的 环境验证开关进行加密，采用加密方式AES

## 源代码

```c
#include <openssl/evp.h>
#include <openssl/err.h>
#include <string.h>
#include <stdio.h>


//错误处理函数
void handleErrors() {
    ERR_print_errors_fp(stderr);
    exit(1);
}

//AES加密函数 (明文数据、明文长度、256-bit密钥、128-bit初始化向量、输入密文缓冲区、输入密文长度)
void encrypt_aes(const unsigned char *plaintext, int plaintext_len,
                const unsigned char *key, const unsigned char *iv,
                unsigned char *ciphertext, int *ciphertext_len) {
    //1.创建加密上下文
    EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
    if (!ctx) handleErrors();

    //2.初始化加密操作(AES-256-CBC模式)
    if (EVP_EncryptInit_ex(ctx, EVP_aes_256_cbc(), NULL, key, iv) != 1)
        handleErrors();

    //3.加密数据 (可能分多次处理)
    if (EVP_EncryptUpdate(ctx, ciphertext, ciphertext_len, plaintext, plaintext_len) != 1)
        handleErrors();

    //4.结束加密（处理填充）
    int len;
    if (EVP_EncryptFinal_ex(ctx, ciphertext + *ciphertext_len, &len) != 1)
        handleErrors();
    *ciphertext_len += len;  //更新总密文长度

    //5.释放上下文
    EVP_CIPHER_CTX_free(ctx);
}

//AES解密函数 (密文数据、密文长度、256-bit密钥、128-bit初始化向量、输入明文缓冲区、输入明文长度)
void decrypt_aes(const unsigned char *ciphertext, int ciphertext_len,
                const unsigned char *key, const unsigned char *iv,
                unsigned char *plaintext, int *plaintext_len) {
    //1.创建解密上下文
    EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
    if (!ctx) handleErrors();

    //2.初始化解密操作
    if (EVP_DecryptInit_ex(ctx, EVP_aes_256_cbc(), NULL, key, iv) != 1)
        handleErrors();

    //3.解密数据
    if (EVP_DecryptUpdate(ctx, plaintext, plaintext_len, ciphertext, ciphertext_len) != 1)
        handleErrors();

    //4.结束解密(去除填充)
    int len;
    if (EVP_DecryptFinal_ex(ctx, plaintext + *plaintext_len, &len) != 1)
        handleErrors();
    *plaintext_len += len; // 更新总明文长度

    //5.释放上下文
    EVP_CIPHER_CTX_free(ctx);
}

// 将二进制数据转换为十六进制字符串
void bin2hex(const unsigned char *bin, int bin_len, char *hex) {
    for (int i = 0; i < bin_len; i++) {
        sprintf(hex + 2 * i, "%02x", bin[i]);
    }
    hex[2 * bin_len] = '\0';
}

// 显示使用帮助
void print_usage(const char *program_name) {
    printf("Usage: %s <text_to_encrypt>\n", program_name);
    printf("Example: %s true\n", program_name);
    printf("Example: %s \"hello world\"\n", program_name);
}

int main(int argc, char *argv[]) {

    // 检查命令行参数
    if (argc != 2) {
        print_usage(argv[0]);
        return 1;
    }

    // 如果用户请求帮助
    if (strcmp(argv[1], "-h") == 0 || strcmp(argv[1], "--help") == 0) {
        print_usage(argv[0]);
        return 0;
    }

    // 获取要加密的文本
    const char *plaintext = argv[1];
    int plaintext_len = strlen(plaintext);

    // 检查输入长度
    if (plaintext_len == 0) {
        printf("Error: Input text cannot be empty\n");
        return 1;
    }

    // 256-bit key (32 bytes) 密钥
    unsigned char key[32] = "0123456789abcdef0123456789abcdef";

    // 128-bit IV (16 bytes)  iv
    unsigned char iv[16] = "1234567890abcdef";


    // 缓冲区（根据输入大小动态计算）
    int ciphertext_buffer_size = plaintext_len + 16; // 预留填充空间
    unsigned char *ciphertext = (unsigned char *)malloc(ciphertext_buffer_size);
    unsigned char *decryptedtext = (unsigned char *)malloc(plaintext_len + 1);

    if (!ciphertext || !decryptedtext) {
        printf("内存分配失败\n");
        free(ciphertext);
        free(decryptedtext);
        return 1;
    }

    int ciphertext_len, decryptedtext_len;

    printf("plaintext: %s\n", plaintext);

    // 加密
    encrypt_aes((unsigned char *)plaintext, plaintext_len, key, iv, ciphertext, &ciphertext_len);

    // 将密文转换为十六进制字符串
    char hex_ciphertext[256] = {0};
    bin2hex(ciphertext, ciphertext_len, hex_ciphertext);

    printf("ciphertext: %s\n", hex_ciphertext);

    // 解密（验证）
    decrypt_aes(ciphertext, ciphertext_len, key, iv, decryptedtext, &decryptedtext_len);
    decryptedtext[decryptedtext_len] = '\0';

    printf("Decryption verification: %s\n", decryptedtext);

    // 清理内存
    free(ciphertext);
    free(decryptedtext);

    return 0;
}
```

## 打包

```sh
gcc encryption.c -o encryption -I "D:/Programs/OpenSSL-Win64/include" "D:/Programs/OpenSSL-Win64//lib/VC/x64/MD/libcrypto.lib"
```

## 使用

```sh
#生成密钥
encryption true
#结果示例
plaintext: true
ciphertext: 38dd3706256f1d66f90165149495e589
Decryption verification: true
```

# MacValidator 的开发与打包

简介：用于验证设备的mac地址，不在列表中则关闭当前项目的启动进程

## Windows版本

### 源代码

```c
#include <windows.h>
#include <iphlpapi.h>
#include <openssl/evp.h>
#include <openssl/err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <jni.h>
// 在文件顶部添加 info 函数的声明
void debug(JNIEnv *env, const char *message);
void info(JNIEnv *env, const char *message);
void error(JNIEnv *env, const char *message);
static const char* AUTHORIZED_MACS[] = {
    "F0-2F-74-DA-48-F4",  // 第一个授权MAC
    "08-BF-B8-86-21-5D",
    NULL
};

void handleErrors() {
    ERR_print_errors_fp(stderr);
    exit(1);
}

void decrypt_aes(const unsigned char *ciphertext, int ciphertext_len,
                 const unsigned char *key, const unsigned char *iv,
                 unsigned char *plaintext, int *plaintext_len) {
    int len;
    EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
    if (!ctx) handleErrors();

    if (EVP_DecryptInit_ex(ctx, EVP_aes_256_cbc(), NULL, key, iv) != 1)
        handleErrors();

    // 初始化plaintext_len为0
    *plaintext_len = 0;

    if (EVP_DecryptUpdate(ctx, plaintext, &len, ciphertext, ciphertext_len) != 1)
        handleErrors();
    *plaintext_len = len;

    if (EVP_DecryptFinal_ex(ctx, plaintext + *plaintext_len, &len) != 1)
        handleErrors();
    *plaintext_len += len;

    EVP_CIPHER_CTX_free(ctx);
}


void getPrimaryMacAddress(char* macStr) {
    PIP_ADAPTER_INFO adapterInfo = NULL;
    ULONG ulOutBufLen = 0;
    DWORD dwRetVal = 0;

    GetAdaptersInfo(NULL, &ulOutBufLen);
    adapterInfo = (IP_ADAPTER_INFO *)malloc(ulOutBufLen);
    if (adapterInfo == NULL) {
        // 如果分配失败，返回空字符串或错误码
        strcpy(macStr, "ERROR");
        return;
    }

    if ((dwRetVal = GetAdaptersInfo(adapterInfo, &ulOutBufLen)) == NO_ERROR) {
        PIP_ADAPTER_INFO adapter = adapterInfo;
        while (adapter) {
            if (adapter->Type != MIB_IF_TYPE_LOOPBACK && adapter->AddressLength == 6) {
                sprintf(macStr, "%02X-%02X-%02X-%02X-%02X-%02X",
                        adapter->Address[0], adapter->Address[1],
                        adapter->Address[2], adapter->Address[3],
                        adapter->Address[4], adapter->Address[5]);
                break;
            }
            adapter = adapter->Next;
        }
    }
    free(adapterInfo);
}

// 通过Java层的日志接口来打印日志
void debug(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger的isDebugEnabled方法
    jmethodID isDebugEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isDebugEnabled", "()Z");
    if (isDebugEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isDebugEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 调用isDebugEnabled方法检查当前日志级别
    jboolean isDebugEnabled = (*env)->CallBooleanMethod(env, logger, isDebugEnabledMethod);
    // 只有在Debug模式下，才输出日志
    if (isDebugEnabled == JNI_TRUE) {
        //printf("Debug logging is enabled.\n");  // Debug输出
        // 获取Logger的debug方法
        jmethodID debugMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "debug", "(Ljava/lang/String;)V");
        if (debugMethod == NULL) {
            fprintf(stderr, "Could not find debug method on Logger, using fallback log\n");
            printf("Log: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的debug方法记录日志
        (*env)->CallVoidMethod(env, logger, debugMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling debug method on Logger\n");
        }
    }
}

void info(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger的isInfoEnabled方法
    jmethodID isInfoEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isInfoEnabled", "()Z");
    if (isInfoEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isInfoEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 调用isInfoEnabled方法检查当前日志级别
    jboolean isInfoEnabled = (*env)->CallBooleanMethod(env, logger, isInfoEnabledMethod);
    // 只有在Info模式下，才输出日志
    if (isInfoEnabled == JNI_TRUE) {
        //printf("Info logging is enabled.\n");  // Debug输出
        // 获取Logger的info方法
        jmethodID infoMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "info", "(Ljava/lang/String;)V");
        if (infoMethod == NULL) {
            fprintf(stderr, "Could not find info method on Logger, using fallback log\n");
            printf("Log: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的debug方法记录日志
        (*env)->CallVoidMethod(env, logger, infoMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling debug method on Logger\n");
        }
    }
}

// 添加一个新的error函数，专门用于记录错误信息
void error(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }
    // 获取Logger的isErrorEnabled方法
    jmethodID isErrorEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isErrorEnabled", "()Z");
    if (isErrorEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isErrorEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }
    jboolean isErrorEnabled = (*env)->CallBooleanMethod(env, logger, isErrorEnabledMethod);

    if(isErrorEnabled == JNI_TRUE) {
        //printf("Error logging is enabled.\n");
         // 获取Logger的error方法
        jmethodID errorMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "error", "(Ljava/lang/String;)V");
        if (errorMethod == NULL) {
            fprintf(stderr, "Could not find error method on Logger, using fallback log\n");
            fprintf(stderr, "ERROR: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的error方法记录日志
        (*env)->CallVoidMethod(env, logger, errorMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling error method on Logger\n");
        }
        // 同时确保错误信息被输出到stderr
        fprintf(stderr, "ERROR: %s\n", message);
    }
}

// 检查MAC地址是否在授权列表中
int isMacAuthorized(const char* currentMac, JNIEnv *env) {
    char logMsg[256];
    snprintf(logMsg, sizeof(logMsg), "Checking MAC: %s", currentMac);
    info(env, logMsg);

    for (int i = 0; AUTHORIZED_MACS[i] != NULL; i++) {
        snprintf(logMsg, sizeof(logMsg), "Authorized MACs: [%d] %s", i, AUTHORIZED_MACS[i]);
        debug(env, logMsg);
        if (_stricmp(AUTHORIZED_MACS[i], currentMac) == 0) {
            //snprintf(logMsg, sizeof(logMsg), "MAC address authorized: %s", AUTHORIZED_MACS[i]);
            //debug(env, logMsg);
            return 1;
        }
    }
    snprintf(logMsg, sizeof(logMsg), "MAC address not found in authorized list");
    info(env, logMsg);
    return 0;
}

__declspec(dllexport) int Java_net_zhiboredian_core_jni_NativeLibraryLoader_initializeAndValidate(JNIEnv *env, jobject obj, jstring enabled) {
    const char *hexStr = (*env)->GetStringUTFChars(env, enabled, NULL);
    if (hexStr == NULL) {
        return 0;
    }

    int ciphertext_len = strlen(hexStr) / 2;
    unsigned char *ciphertext = (unsigned char *)malloc(ciphertext_len);
    if (ciphertext == NULL) {
        (*env)->ReleaseStringUTFChars(env, enabled, hexStr);
        return 0;
    }

    for (int i = 0; i < ciphertext_len; i++) {
        sscanf(hexStr + 2*i, "%02hhx", &ciphertext[i]);
    }
    (*env)->ReleaseStringUTFChars(env, enabled, hexStr);

    unsigned char key[32] = "0123456789abcdef0123456789abcdef";
    unsigned char iv[16] = "1234567890abcdef";

    unsigned char *decryptedtext = (unsigned char *)malloc(ciphertext_len + 1);
    if (decryptedtext == NULL) {
        free(ciphertext);
        return 0;
    }

    int decryptedtext_len = 0;
    decrypt_aes(ciphertext, ciphertext_len, key, iv, decryptedtext, &decryptedtext_len);
    decryptedtext[decryptedtext_len] = '\0';

    // 输出解密结果
    char logMsg[256];
    snprintf(logMsg, sizeof(logMsg), "Decrypted Text: %s", decryptedtext);
    debug(env, logMsg);

    int shouldValidateMac = 0;
    if (strcmp((const char*)decryptedtext, "true") == 0) {
        shouldValidateMac = 1;
        snprintf(logMsg, sizeof(logMsg), "MAC validation is required");
        info(env, logMsg);
    } else if (strcmp((const char*)decryptedtext, "false") == 0) {
        snprintf(logMsg, sizeof(logMsg), "Skipping MAC validation");
        info(env, logMsg);
    } else {
        snprintf(logMsg, sizeof(logMsg), "Invalid decrypted value: %s, exiting program", decryptedtext);
        error(env, logMsg);
        free(ciphertext);
        free(decryptedtext);
        ExitProcess(1); // Terminate program
        return 0;
    }

    free(ciphertext);
    free(decryptedtext);

    char currentMac[18] = {0};
    getPrimaryMacAddress(currentMac);

    // 如果需要验证MAC
    if (shouldValidateMac) {
        if (!isMacAuthorized(currentMac, env)) {
            snprintf(logMsg, sizeof(logMsg), "MAC validation failed, exiting program");
            error(env, logMsg);
            ExitProcess(1); // Terminate program
            return 0;
        }
        snprintf(logMsg, sizeof(logMsg), "MAC validation passed");
        info(env, logMsg);
    } else {
        snprintf(logMsg, sizeof(logMsg), "MAC validation skipped");
        info(env, logMsg);
    }

    return 1; // Successful validation
}

```

### 打包

```sh
gcc -I "D:/Program Files/Java/jdk-1.8/include" -I "D:/Program Files/Java/jdk-1.8/include/win32" -I "D:/Programs/OpenSSL-Win64/include" -I "D:/Programs/OpenSSL-Win64/include/openssl" -L "D:/Programs/OpenSSL-Win64/lib" -o MacValidator.dll MacValidator.c "D:/Programs/OpenSSL-Win64/lib/VC/x64/MD/libssl.lib" "D:/Programs/OpenSSL-Win64/lib/VC/x64/MD/libcrypto.lib" -liphlpapi -shared
```

重点1：安装OpenSSL

重点2：指定明确的openssl路径

## Linux版本

### 源代码

```c
#include <windows.h>
#include <iphlpapi.h>
#include <openssl/evp.h>
#include <openssl/err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <jni.h>
// 在文件顶部添加 info 函数的声明
void debug(JNIEnv *env, const char *message);
void info(JNIEnv *env, const char *message);
void error(JNIEnv *env, const char *message);
static const char* AUTHORIZED_MACS[] = {
    "F0-2F-74-DA-48-F4",  // 第一个授权MAC
    "08-BF-B8-86-21-5D",
    NULL
};

void handleErrors() {
    ERR_print_errors_fp(stderr);
    exit(1);
}

void decrypt_aes(const unsigned char *ciphertext, int ciphertext_len,
                 const unsigned char *key, const unsigned char *iv,
                 unsigned char *plaintext, int *plaintext_len) {
    int len;
    EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
    if (!ctx) handleErrors();

    if (EVP_DecryptInit_ex(ctx, EVP_aes_256_cbc(), NULL, key, iv) != 1)
        handleErrors();

    // 初始化plaintext_len为0
    *plaintext_len = 0;

    if (EVP_DecryptUpdate(ctx, plaintext, &len, ciphertext, ciphertext_len) != 1)
        handleErrors();
    *plaintext_len = len;

    if (EVP_DecryptFinal_ex(ctx, plaintext + *plaintext_len, &len) != 1)
        handleErrors();
    *plaintext_len += len;

    EVP_CIPHER_CTX_free(ctx);
}


void getPrimaryMacAddress(char* macStr) {
    PIP_ADAPTER_INFO adapterInfo = NULL;
    ULONG ulOutBufLen = 0;
    DWORD dwRetVal = 0;

    GetAdaptersInfo(NULL, &ulOutBufLen);
    adapterInfo = (IP_ADAPTER_INFO *)malloc(ulOutBufLen);
    if (adapterInfo == NULL) {
        // 如果分配失败，返回空字符串或错误码
        strcpy(macStr, "ERROR");
        return;
    }

    if ((dwRetVal = GetAdaptersInfo(adapterInfo, &ulOutBufLen)) == NO_ERROR) {
        PIP_ADAPTER_INFO adapter = adapterInfo;
        while (adapter) {
            if (adapter->Type != MIB_IF_TYPE_LOOPBACK && adapter->AddressLength == 6) {
                sprintf(macStr, "%02X-%02X-%02X-%02X-%02X-%02X",
                        adapter->Address[0], adapter->Address[1],
                        adapter->Address[2], adapter->Address[3],
                        adapter->Address[4], adapter->Address[5]);
                break;
            }
            adapter = adapter->Next;
        }
    }
    free(adapterInfo);
}

// 通过Java层的日志接口来打印日志
void debug(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger的isDebugEnabled方法
    jmethodID isDebugEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isDebugEnabled", "()Z");
    if (isDebugEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isDebugEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 调用isDebugEnabled方法检查当前日志级别
    jboolean isDebugEnabled = (*env)->CallBooleanMethod(env, logger, isDebugEnabledMethod);
    // 只有在Debug模式下，才输出日志
    if (isDebugEnabled == JNI_TRUE) {
        //printf("Debug logging is enabled.\n");  // Debug输出
        // 获取Logger的debug方法
        jmethodID debugMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "debug", "(Ljava/lang/String;)V");
        if (debugMethod == NULL) {
            fprintf(stderr, "Could not find debug method on Logger, using fallback log\n");
            printf("Log: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的debug方法记录日志
        (*env)->CallVoidMethod(env, logger, debugMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling debug method on Logger\n");
        }
    }
}

void info(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 获取Logger的isInfoEnabled方法
    jmethodID isInfoEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isInfoEnabled", "()Z");
    if (isInfoEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isInfoEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }

    // 调用isInfoEnabled方法检查当前日志级别
    jboolean isInfoEnabled = (*env)->CallBooleanMethod(env, logger, isInfoEnabledMethod);
    // 只有在Info模式下，才输出日志
    if (isInfoEnabled == JNI_TRUE) {
        //printf("Info logging is enabled.\n");  // Debug输出
        // 获取Logger的info方法
        jmethodID infoMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "info", "(Ljava/lang/String;)V");
        if (infoMethod == NULL) {
            fprintf(stderr, "Could not find info method on Logger, using fallback log\n");
            printf("Log: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的debug方法记录日志
        (*env)->CallVoidMethod(env, logger, infoMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling debug method on Logger\n");
        }
    }
}

// 添加一个新的error函数，专门用于记录错误信息
void error(JNIEnv *env, const char *message) {
    // 查找SLF4J的LoggerFactory类
    jclass loggerFactoryClass = (*env)->FindClass(env, "org/slf4j/LoggerFactory");
    if (loggerFactoryClass == NULL) {
        fprintf(stderr, "Could not find LoggerFactory class, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }

    // 获取LoggerFactory.getLogger方法
    jmethodID getLoggerMethod = (*env)->GetStaticMethodID(env, loggerFactoryClass, "getLogger", "(Ljava/lang/Class;)Lorg/slf4j/Logger;");
    if (getLoggerMethod == NULL) {
        fprintf(stderr, "Could not find getLogger method, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }

    // 获取Logger对象
    jclass classForLogger = (*env)->FindClass(env, "net/zhiboredian/core/jni/NativeLibraryLoader");
    jobject logger = (*env)->CallStaticObjectMethod(env, loggerFactoryClass, getLoggerMethod, classForLogger);

    if (logger == NULL) {
        fprintf(stderr, "Could not get Logger instance, using fallback log\n");
        fprintf(stderr, "ERROR: %s\n", message);
        return;
    }
    // 获取Logger的isErrorEnabled方法
    jmethodID isErrorEnabledMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "isErrorEnabled", "()Z");
    if (isErrorEnabledMethod == NULL) {
        fprintf(stderr, "Could not find isErrorEnabled method on Logger, using fallback log\n");
        printf("Log: %s\n", message);
        return;
    }
    jboolean isErrorEnabled = (*env)->CallBooleanMethod(env, logger, isErrorEnabledMethod);

    if(isErrorEnabled == JNI_TRUE) {
        //printf("Error logging is enabled.\n");
         // 获取Logger的error方法
        jmethodID errorMethod = (*env)->GetMethodID(env, (*env)->GetObjectClass(env, logger), "error", "(Ljava/lang/String;)V");
        if (errorMethod == NULL) {
            fprintf(stderr, "Could not find error method on Logger, using fallback log\n");
            fprintf(stderr, "ERROR: %s\n", message);
            return;
        }

        // 创建Java字符串用于日志信息
        jstring info = (*env)->NewStringUTF(env, message);

        // 调用Logger的error方法记录日志
        (*env)->CallVoidMethod(env, logger, errorMethod, info);

        // 检查JNI异常
        if ((*env)->ExceptionCheck(env)) {
            (*env)->ExceptionDescribe(env);
            (*env)->ExceptionClear(env);
            fprintf(stderr, "Error calling error method on Logger\n");
        }
        // 同时确保错误信息被输出到stderr
        fprintf(stderr, "ERROR: %s\n", message);
    }
}

// 检查MAC地址是否在授权列表中
int isMacAuthorized(const char* currentMac, JNIEnv *env) {
    char logMsg[256];
    snprintf(logMsg, sizeof(logMsg), "Checking MAC: %s", currentMac);
    info(env, logMsg);

    for (int i = 0; AUTHORIZED_MACS[i] != NULL; i++) {
        snprintf(logMsg, sizeof(logMsg), "Authorized MACs: [%d] %s", i, AUTHORIZED_MACS[i]);
        debug(env, logMsg);
        if (_stricmp(AUTHORIZED_MACS[i], currentMac) == 0) {
            //snprintf(logMsg, sizeof(logMsg), "MAC address authorized: %s", AUTHORIZED_MACS[i]);
            //debug(env, logMsg);
            return 1;
        }
    }
    snprintf(logMsg, sizeof(logMsg), "MAC address not found in authorized list");
    info(env, logMsg);
    return 0;
}

__declspec(dllexport) int Java_net_zhiboredian_core_jni_NativeLibraryLoader_initializeAndValidate(JNIEnv *env, jobject obj, jstring enabled) {
    const char *hexStr = (*env)->GetStringUTFChars(env, enabled, NULL);
    if (hexStr == NULL) {
        return 0;
    }

    int ciphertext_len = strlen(hexStr) / 2;
    unsigned char *ciphertext = (unsigned char *)malloc(ciphertext_len);
    if (ciphertext == NULL) {
        (*env)->ReleaseStringUTFChars(env, enabled, hexStr);
        return 0;
    }

    for (int i = 0; i < ciphertext_len; i++) {
        sscanf(hexStr + 2*i, "%02hhx", &ciphertext[i]);
    }
    (*env)->ReleaseStringUTFChars(env, enabled, hexStr);

    unsigned char key[32] = "0123456789abcdef0123456789abcdef";
    unsigned char iv[16] = "1234567890abcdef";

    unsigned char *decryptedtext = (unsigned char *)malloc(ciphertext_len + 1);
    if (decryptedtext == NULL) {
        free(ciphertext);
        return 0;
    }

    int decryptedtext_len = 0;
    decrypt_aes(ciphertext, ciphertext_len, key, iv, decryptedtext, &decryptedtext_len);
    decryptedtext[decryptedtext_len] = '\0';

    // 输出解密结果
    char logMsg[256];
    snprintf(logMsg, sizeof(logMsg), "Decrypted Text: %s", decryptedtext);
    debug(env, logMsg);

    int shouldValidateMac = 0;
    if (strcmp((const char*)decryptedtext, "true") == 0) {
        shouldValidateMac = 1;
        snprintf(logMsg, sizeof(logMsg), "MAC validation is required");
        info(env, logMsg);
    } else if (strcmp((const char*)decryptedtext, "false") == 0) {
        snprintf(logMsg, sizeof(logMsg), "Skipping MAC validation");
        info(env, logMsg);
    } else {
        snprintf(logMsg, sizeof(logMsg), "Invalid decrypted value: %s, exiting program", decryptedtext);
        error(env, logMsg);
        free(ciphertext);
        free(decryptedtext);
        ExitProcess(1); // Terminate program
        return 0;
    }

    free(ciphertext);
    free(decryptedtext);

    char currentMac[18] = {0};
    getPrimaryMacAddress(currentMac);

    // 如果需要验证MAC
    if (shouldValidateMac) {
        if (!isMacAuthorized(currentMac, env)) {
            snprintf(logMsg, sizeof(logMsg), "MAC validation failed, exiting program");
            error(env, logMsg);
            ExitProcess(1); // Terminate program
            return 0;
        }
        snprintf(logMsg, sizeof(logMsg), "MAC validation passed");
        info(env, logMsg);
    } else {
        snprintf(logMsg, sizeof(logMsg), "MAC validation skipped");
        info(env, logMsg);
    }

    return 1; // Successful validation
}

```

### 打包

```sh
sudo apt install openjdk-8-jdk
sudo apt install libssl-dev

gcc -I/usr/lib/jvm/java-8-openjdk-amd64/include -I/usr/lib/jvm/java-8-openjdk-amd64/include/linux -fPIC -shared -o validator.so validatorForLinux.c -lssl -lcrypto
```



## java中的调用

### 第一步：启动验证

EnvironmentValidator.java

```java
@Configuration
public class EnvironmentValidator {
    @Value("${core.environment.validation}")
    private String enabled;
    @PostConstruct
    public void validateOnStartup() {
        try {
            NativeLibraryLoader.initializeAndValidate(enabled);
            System.out.println("DLL验证通过，应用启动继续...");
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("无法加载验证库(MacValidator.dll)", e);
        }
    }
}
```

### 第二步：JNI加载动态链接库文件

NativeLibraryLoader.java

```java
@Component
public class NativeLibraryLoader {
    private static final String NATIVE_DIR = "native";
    @Value("${core.native.library:}")
    private String configuredPath;
    @Value("${core.environment.validation:}")
    private String validation;
    @PostConstruct
    public void loadLibrary() {
        try {
            String osName = System.getProperty("os.name").toLowerCase();
            System.out.printf("Loading native library for OS: %s%n", osName);
            System.out.printf("Configured path: %s, Validation: %s%n", configuredPath, validation);

            // 确定库文件名和后缀
            LibraryInfo libraryInfo = determineLibraryInfo(osName);
            if (libraryInfo == null) {
                System.err.println("Unsupported operating system: " + osName);
                return;
            }

            // 1. 首先尝试从配置路径加载
            Path libraryPath = findLibraryFromConfiguredPath(libraryInfo.fileName);

            if (libraryPath != null && Files.exists(libraryPath)) {
                // 从文件系统路径加载
                System.load(libraryPath.toAbsolutePath().toString());
                System.out.println("Successfully loaded library from: " + libraryPath);
            } else {
                // 2. 配置路径找不到，从类路径加载
                loadFromClasspath(libraryInfo.fileName, libraryInfo.tempSuffix);
            }

        } catch (Exception e) {
            System.err.println("Failed to load native library: " + e.getMessage());
            e.printStackTrace();
            System.exit(1);
        }
    }

    /**
     * 根据操作系统确定库文件信息
     */
    private LibraryInfo determineLibraryInfo(String osName) {
        if (osName.contains("windows")) {
            return new LibraryInfo("MacValidator.dll", ".dll");
        } else if (osName.contains("linux")) {
            return new LibraryInfo("validator.so", ".so");
        }
        return null;
    }

    /**
     * 从配置的路径查找库文件
     */
    private Path findLibraryFromConfiguredPath(String expectedFileName) {
        // 如果没有配置路径，直接返回null
        if (configuredPath == null || configuredPath.trim().isEmpty()) {
            System.out.println("No configured path specified, will try classpath");
            return null;
        }

        String configPath = configuredPath.trim();
        System.out.println("Searching in configured path: " + configPath);

        try {
            Path basePath = Paths.get(configPath);

            // 检查配置的路径是否已经是一个文件（包含文件名）
            if (Files.isRegularFile(basePath)) {
                System.out.println("Found library file directly: " + basePath);
                return basePath;
            }

            // 检查配置的路径是否是一个目录，需要拼接文件名
            if (Files.isDirectory(basePath)) {
                Path libraryPath = basePath.resolve(expectedFileName);
                if (Files.isRegularFile(libraryPath)) {
                    System.out.println("Found library in directory: " + libraryPath);
                    return libraryPath;
                } else {
                    System.out.println("Library not found in directory: " + libraryPath);
                    return null;
                }
            }

            // 如果配置的路径既不是文件也不是目录，尝试解析为相对路径
            String projectRoot = System.getProperty("user.dir");
            Path relativePath = Paths.get(projectRoot, configPath);

            if (Files.isRegularFile(relativePath)) {
                System.out.println("Found library at relative path: " + relativePath);
                return relativePath;
            }

            // 尝试相对路径+文件名
            Path relativePathWithFileName = Paths.get(projectRoot, configPath, expectedFileName);
            if (Files.isRegularFile(relativePathWithFileName)) {
                System.out.println("Found library at relative path with filename: " + relativePathWithFileName);
                return relativePathWithFileName;
            }

            System.out.println("Library not found in configured path");
            return null;

        } catch (Exception e) {
            System.err.println("Error processing configured path: " + configPath + ", error: " + e.getMessage());
            return null;
        }
    }

    /**
     * 从类路径加载库文件
     */
    private void loadFromClasspath(String libraryFileName, String tempSuffix) throws IOException {
        String resourcePath = NATIVE_DIR + "/" + libraryFileName;
        System.out.println("Trying to load from classpath: " + resourcePath);

        try (InputStream resourceStream = getClass().getClassLoader().getResourceAsStream(resourcePath)) {
            if (resourceStream == null) {
                throw new IOException("Could not find native library in classpath: " + resourcePath +
                        ". Please check if the library file exists in the resources/native/ directory.");
            }

            // 创建临时文件来加载库
            Path tempFile = Files.createTempFile("native-lib", tempSuffix);
            Files.copy(resourceStream, tempFile, StandardCopyOption.REPLACE_EXISTING);

            // 加载临时文件中的库
            System.load(tempFile.toAbsolutePath().toString());
            System.out.println("Successfully loaded library from classpath: " + resourcePath);

            // 设置临时文件在JVM退出时自动删除
            tempFile.toFile().deleteOnExit();
        }
    }

    // 声明native方法
    public static native void initializeAndValidate(String validation);

    /**
     * 库文件信息封装类
     */
    private static class LibraryInfo {
        final String fileName;
        final String tempSuffix;

        LibraryInfo(String fileName, String tempSuffix) {
            this.fileName = fileName;
            this.tempSuffix = tempSuffix;
        }
    }
}
```

