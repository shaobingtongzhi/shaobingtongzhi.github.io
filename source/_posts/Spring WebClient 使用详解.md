---
title: Spring WebClient 使用详解
date: 2025-04-15 10:00:00
tags:
  - Java
  - Spring Boot
  - WebClient
  - HTTP
categories:
  - Java开发
---

> 本文全面介绍了 Spring WebClient 的使用方式，包括 GET/POST 请求、上传文件、处理异常、自定义配置等内容，是 `RestTemplate` 的现代替代方案，适用于微服务与高并发场景。


## 📌 WebClient 简介

WebClient 是 Spring 5 中引入的一个 **非阻塞式 HTTP 客户端**，用于替代传统的 `RestTemplate`。

- ✅ 支持同步 / 异步请求
- ✅ 支持响应式编程（基于 Project Reactor）
- ✅ 支持流式处理、文件上传、异常处理等
- ✅ 在 Spring Boot 和 Spring Cloud 项目中非常常用

---

## 🚀 快速开始
WebClient 的典型调用链如下：
```java
WebClient.create() // 创建 WebClient 实例
    .method(HttpMethod.GET) // 请求方法（也可以用 .get(), .post() 等快捷方式）
    .uri("https://api.example.com/data") // 设置 URL
    .headers(headers -> { ... }) // 设置请求头（可选）
    .body(...) // 设置请求体（可选，通常用于 POST）
    .retrieve() // 发起请求
    .bodyToMono(...) // 处理响应（转换为 Mono、Flux、对象等）
    .block(); // 获取响应结果（阻塞）

```
也可以简写为

```java
WebClient webClient = WebClient.create();

String result = webClient.get()
        .uri("https://api.example.com/data")
        .retrieve()
        .bodyToMono(String.class)
        .block();  // 同步阻塞获取结果

System.out.println(result);
```
## 🔧 POST 请求示例
```java
String json = "{\"name\":\"张三\",\"age\":25}";

String response = WebClient.create()
        .post()
        .uri("https://api.example.com/user")
        .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .bodyValue(json)
        .retrieve()
        .bodyToMono(String.class)
        .block();
```
## ✅ 请求构建部分详解

### 1. 创建 WebClient 实例
```java
WebClient webClient = WebClient.create(); // 默认实例
```
或者自定义
```java
WebClient webClient = WebClient.builder()
        .baseUrl("https://api.example.com")
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .build();
```
### 2. 请求方法
```java
webClient.get()     // GET 请求
webClient.post()    // POST 请求
webClient.put()     // PUT 请求
webClient.delete()  // DELETE 请求
webClient.method(HttpMethod.OPTIONS) // 更灵活的方式

```
### 3. 设置URI
```java
.uri("https://api.example.com/data")
.uri(uriBuilder -> uriBuilder
     .path("/users")
     .queryParam("id", 123)
     .build())
```
### 4. 设置请求头（Headers）
```java
.headers(headers -> {
    headers.set("Authorization", "Bearer xxx");
    headers.setContentType(MediaType.APPLICATION_JSON);
})
```
也可以单独设置
```java
.header("Custom-Header", "value")
```
### 5. 设置请求体（body）
a. 使用 bodyValue(...) 传入一个对象
```java
.bodyValue(new User("张三", 18))
```
b. 使用 body(...) 搭配 Publisher
```java
.body(Mono.just(new User("张三", 18)), User.class)
```
c. 文件上传用 MultipartBodyBuilder
```java
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("file", new FileSystemResource(new File("path.txt")));

.body(BodyInserters.fromMultipartData(builder.build()))

```
---
## 🔄 响应处理部分详解
### 1. retrieve() 方法
- 表示发出请求并准备接收响应

- 默认只处理 2xx 成功响应，其他状态会抛异常

- 如果你只关心 body 内容，推荐用这个

```java
.retrieve()
```
### 2. exchangeToMono() / exchangeToFlux()

- 获取完整响应（包括 headers / status / cookies）

- 适用于你要自定义错误处理逻辑时
```java
.exchangeToMono(response -> {
    if (response.statusCode().is2xxSuccessful()) {
        return response.bodyToMono(String.class);
    } else {
        return response.createException().flatMap(Mono::error);
    }
});
```
### 3. 提取响应体（响应体转换）
```java
.bodyToMono(String.class)      // 响应体是一个对象（单个）
.bodyToMono(MyDto.class)       // 自动反序列化为自定义对象
.bodyToFlux(MyDto.class)       // 响应体是数组 / 流时使用

```
## ⚠️ 异常处理详解
```java
webClient.get()
    .uri(url)
    .retrieve()
    .onStatus(status -> status.is4xxClientError(), response -> {
    	return Mono.error(new RuntimeException("客户端错误"));
    })
    .onStatus(status -> status.is5xxServerError(), response -> {
    	return Mono.error(new RuntimeException("服务端错误"));
    })
    .bodyToMono(responseType)
    .block(); // 阻塞获取结果
```
或者使用 exchangeToMono 自己解析状态码。

## ⏹ 阻塞或非阻塞获取结果
### 阻塞式（常用于业务调用）
```java
.block()              // 获取 Mono 最终结果
.block(Duration.ofSeconds(5)) // 设置超时
```
### 非阻塞式（用于响应式/异步）
```java
.subscribe(result -> {
    System.out.println("结果是: " + result);
});
```
## 🌈 进阶功能（可选）
### 设置 Cookies
```java
.cookie("token", "123456")
```

### 设置 Basic Auth
```java
WebClient.builder()
    .defaultHeaders(headers -> headers.setBasicAuth("user", "password"))
    .build();
```
### 设置连接超时、读取超时（Reactor Netty 配置）
```java
HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(3));

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();

```

## ✅ 常用方法结构参考表
| 分类 | 方法 | 说明 |
| ---- | --------- | ------------ |
|实例创建	|WebClient.create()	|创建默认实例|
|           |WebClient.builder()	|自定义实例||
|请求方法	|.get(), .post() 等	|快捷设置请求方法|
|URI		|.uri(...)	|支持字符串或 URI 构建器|
|Headers	|.header(), .headers()	|设置请求头|
|Body		|.bodyValue(), .body()	|设置请求体|
|发起请求	|.retrieve(), .exchangeToMono()	|发出请求并处理响应|
|响应处理	|.bodyToMono(), .bodyToFlux()	|转换响应体类型|
|异常处理	|.onStatus(...)	|根据状态码处理错误|
|获取结果	|.block(), .subscribe()	|阻塞/非阻塞获取结果|


## ⚙️ 自定义 WebClient（添加 Header 和 BaseUrl）
```java
WebClient webClient = WebClient.builder()
        .baseUrl("https://api.example.com")
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .build();
```

## ⏱ 异步调用（非阻塞式）
```java
WebClient webClient = WebClient.create();

Mono<String> resultMono = webClient.get()
        .uri("https://api.example.com/data")
        .retrieve()
        .bodyToMono(String.class);

resultMono.subscribe(result -> {
    System.out.println("异步结果: " + result);
});
```

## 📤 上传文件 Multipart 示例
```java
MultiValueMap<String, Object> formData = new LinkedMultiValueMap<>();
formData.add("file", new FileSystemResource(new File("path/to/file.txt")));

String response = WebClient.create()
        .post()
        .uri("https://api.example.com/upload")
        .contentType(MediaType.MULTIPART_FORM_DATA)
        .bodyValue(formData)
        .retrieve()
        .bodyToMono(String.class)
        .block();
```
## 🔁 WebClient vs RestTemplate 对比

| 特性 | WebClient | RestTemplate |
| ---- | --------- | ------------ |
|同步请求|✅ 支持|✅ 支持|
|异步请求	|✅ 支持	|❌ 不支持|
|响应式支持	|✅ 完全支持	|❌ 不支持|
|Spring 推荐	|✅ 推荐使用	|❌ 已标记为弃用|
|高并发场景适用	|✅ 非常适合	|❌ 不适合|

## ✅ 总结
WebClient 是现代 Java 后端进行 HTTP 通信的推荐方式，尤其适合：

- 微服务之间的调用

- AI/中台服务的数据交互

- 高并发与异步处理场景

- 对响应时间有更高要求的应用

- 建议新项目统一采用 WebClient，逐步替代旧有的 RestTemplate 实现。