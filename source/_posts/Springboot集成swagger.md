---
title: Springboot 集成 swagger
date: 2024-04-18
categories:
  - 学习笔记
  - 01-Spring学习笔记
tags:
  - Springboot
  - 接口文档生成
  - swagger
---





# 引入

pom.xml

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

# 增加配置类

配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Value("${swagger.title}")
    private String title;
    @Value("${swagger.description}")
    private String description;
    @Value("${swagger.version}")
    private String version;
    @Value("${swagger.basePackage}")
    private String basePackage;

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage(basePackage)) // 指定扫描的controller路径
                .paths(PathSelectors.any())
                .build().apiInfo(apiInfo());
    }
    private ApiInfo apiInfo(){
        return new ApiInfo(title,
                description,
                version,
                "",
                new Contact("", "", ""),
                "",
                "",
                new ArrayList());
    }
}
```

配置文件

```yml
swagger:
  title: 智博综合能源智慧管理平台
  description: 平台接口文档
  version: 1.0.0
  basePackage: net.zhiboredian.controller # 指定扫描的controller路径
```

# 常用注解

**@Api**

将类标记为Swagger资源。默认情况下，Swagger Core将只包括和内省使用@Api注释的类，并将忽略其他资源（JAX-RS端点、Servlet等）

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/Snipaste_2024-04-25_09-30-47.1acl5wosz9r4.webp)

**@Tag**

注释可以应用于类或方法

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/Snipaste_2024-04-25_09-31-06.imy7z9duuqg.webp)

**@ApiOperation**

描述针对特定路径的操作，通常是HTTP方法。
具有等效路径的操作被分组在单个操作对象中。HTTP方法和路径的组合创建了一个唯一的操作

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/Snipaste_2024-04-25_09-36-33.1044vq08xkfk.webp)

**@ApiParam**

为操作参数添加其他元数据

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/Snipaste_2024-04-25_09-38-40.1fxy30ztvxi8.webp)

**@ApiModelProperty**

添加和操作模型特性的数据。

![](https://cdn.jsdelivr.net/gh/hfshaobing/picx-images-hosting@master/Snipaste_2024-04-25_09-40-14.6a9cpp1oz9s0.webp)

# 报错处理

1.  Failed to start bean 'documentationPluginsBootstrapper'

参考链接：https://blog.csdn.net/weixin_49523761/article/details/122305980

参考链接：https://blog.csdn.net/qiziyu520/article/details/123685295

方案一：添加配置项（治标）

```yml
spring:
	mvc:
    	pathmatch:
      		matching-strategy: ant_path_matcher
```

方案二：在项目里添加这个 bean ：（加在配置类里就可）(治本)，没试

```java
@Bean
public static BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
    return new BeanPostProcessor() {

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                customizeSpringfoxHandlerMappings(getHandlerMappings(bean));
            }
            return bean;
        }

        private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
            List<T> copy = mappings.stream()
                    .filter(mapping -> mapping.getPatternParser() == null)
                    .collect(Collectors.toList());
            mappings.clear();
            mappings.addAll(copy);
        }

        @SuppressWarnings("unchecked")
        private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
            try {
                Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                field.setAccessible(true);
                return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
            } catch (IllegalArgumentException | IllegalAccessException e) {
                throw new IllegalStateException(e);
            }
        }
    };
}
```

2. 修改swagger的日志级别，用来屏蔽"swagger中整型参数无法设置默认值WARN提示"问题

```yml
logging:
  file:
    name: log/log.log
  level:
    root: info
    net.zhiboredian : info
    io:
      swagger: error #将swagger的日志级别调试为error，用来屏蔽"swagger中整型参数无法设置默认值WARN提示"问题
```

