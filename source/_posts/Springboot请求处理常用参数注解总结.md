---
title: Springboot 请求处理常用参数注解总结
date: 2024-04-12
categories:
  - 学习笔记
  - 01-Spring学习笔记
---
注意：GET请求一般采用 RequestParam 接收，POST请求一般采用 RequestBody 接收，前端传参时，设置ContentType = "application/json"

# 路径变量

@PathVariable

```java
//请求地址：/car/3/name/zhangsan  =>  /car/{id}/name/{uname}
@GetMapping("/car/{id}/name/{uname}")
public Map<String,Object> index(@PathVariable("id") Integer id,
                                @PathVariable("name") String name,
                                @PathVariable Map<String,String> pv
                               ){
    Map<String, Object> map = new HashMap<>();
    map.put("id",id);
    map.put("name",name);
    map.put("pv",pv);
    return map;
}
```

# 获取请求头

@RequestHeader

```java
//请求地址：/test1
@GetMapping("/test1")
public Map test1(@RequestHeader("User-Agent") String userAgent,
                 @RequestHeader Map<String,String> header
                ){
    HashMap<String, Object> map = new HashMap<>();
    map.put("userAgent",userAgent);
    map.put("header",header);
    return map;
}
```

# 获取请求参数

@RequestParam

```java
//请求地址：/testRequestParam?a=1&b=2&love=篮球&love=羽毛球
@GetMapping("/testRequestParam")
public Map testRequestParam(@RequestParam("a") Integer a,
                            @RequestParam("b") String b,
                            @RequestParam("love") List<String> love,
                            @RequestParam Map<String,String> params
                           ){
    HashMap<String, Object> map = new HashMap<>();
    map.put("a",a);
    map.put("b",b);
    map.put("love",love);
    map.put("params",params);
    //注意 params 里的 love 是不行的
    return map;
}
```

# 获取Cookie值

@CookieValue

```java
@GetMapping("/testCookieValue")
public Map testCookieValue(
    @CookieValue("Hm_lvt_85fad12bb9a6dab448f4eff0a19299a5") String cookieValue,
    @CookieValue("Hm_lvt_85fad12bb9a6dab448f4eff0a19299a5") Cookie cookie){
    HashMap<String, Object> map = new HashMap<>();
    map.put("cookieValue",cookieValue);
    map.put("cookie",cookie);
    return map;
}
```

# 获取Request域属性

@RequestAttribute

```java
//使用场景，a请求跳转b请求
```

# 获取请求体

@RequestBody

```java
//注意：只有POST请求才有请求体，且请求头中的 Content-Type: application/x-www-form-urlencoded
@PostMapping("/testRequestBody")
public Map testRequestBody(@RequestBody String content){
    HashMap<String, Object> map = new HashMap<>();
    map.put("content",content);
    return map;
}

```

# 矩阵变量

@MatrixVariable

```java
//使用场景：浏览器禁用cookie后，无法获得服务器端的session_id
```





