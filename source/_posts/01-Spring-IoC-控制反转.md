---
title: 01.Spring-IoC 控制反转
date: 2023-04-26 19:19:48
tags: 
	- Spring
	- IoC
	- 控制反转
categories:
	- 学习笔记
	- 01-Spring学习笔记
---
# Spring IoC控制反转

## 概念

IoC作用：Spring 通过IoC 容器来管理所有 Java 对象的实例化和初始化，控制对象与对象之间的依赖关系
> 我们将由 IoC 容器管理的 Java 对象称为 Spring Bean，它与使用关键字 new 创建的 Java 对象没有任何区别

## 依赖注入

DI (Dependency Injection) 依赖注入

IoC 是思想，DI 是实现

概念： 指Spring创建对象的过程中，将对象依赖属性通过配置进行注入

### 基于XML管理bean

#### 入门案例

TestUser.java

```java
//1.加载Spring配置文件
//补充：
//ClassPathXmlApplicationContext 通过读取类路径下的 XML 格式的配置文件创建 IoC 容器对象
//FileSystemXmlApplicationContext 通过文件系统路径读取 XML 格式的配置文件创建 IoC 容器对象
//ConfigureableApplicationContext ApplicationContext的子接口，包含一些扩展方法 refresh() 和 close(),让 ApplicationContext 具有启动、关闭和刷新上下文的能力
//WebApplicationContext 专门为Web应用准备，基于 Web 环境创建 IoC 容器对象，并将对象引入存入 ServletContext 域中
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
//2.创建对象
User user = (User) context.getBean("user");
//3.接着就可以调用类的方法了
user.add();
```
beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
        完成user对象创建
        1.bean标签
            id:唯一标识
            class:要创建所在类的全路径
    -->
    <bean id="user" class="com.example.spring.User"></bean>
</beans>
```
User.java
```java
package com.example.spring;
public class User {
    //无参数构造
    public User(){
        System.out.println("0：无参数构造执行");
    }
    public void add(){
        System.out.println("add......");
    }
}

```
#### 获取bean的三种方式

- 根据ID获取
```java
User user = (User) context.getBean("user");
```
- 根据class获取
```java
User user = context.getBean(User.class);
```
- 根据id和class获取
```java
User user = context.getBean("user",User.class);
```
**扩展知识**

1. 如果组件类实现了接口，根据接口类型可以获取bean吗? 可以
2. 如果一个接口有多个实现类，这些实现类都配置了bean，那么根据接口类型可以获取bean吗？ 不可以，因为bean不唯一

#### 注入方式

原生注入方法：

```java
//方式一
Book book = new Book();
book.setBname("java web学习指南");
book.setAuthor("中国电子科技大学");
//方式二
Book book = new Book("java web学习指南","中国电子科技大学");

```
- 第一种：set注入
```xml
<bean id="book" class="com.example.spring.iocxml.di.Book">
    <property name="bname" value="java web学习指南"></property>
    <property name="author" value="中国电子科技大学"></property>
</bean>
```

- 第二种：构造注入
```xml
<bean id="bookCon" class="com.example.spring.iocxml.di.Book">
    <constructor-arg name="bname" value="java web学习指南"></constructor-arg>
    <constructor-arg name="author" value="中国电子"></constructor-arg>
</bean>
```

#### 特殊属性类型注入

- 对象类型属性注入 关键字 ref

```xml
<property name="dept" ref="dept">
<!--<ref bean="dept"></ref>-->
</property>
```

​    

- 数组类型属性注入 关键字 array

```xml
<property name="loves">
<array>
<value>游泳</value>
<value>打球</value>
<value>吃饭</value>
</array>
</property>
```

​    

- 集合类型属性注入
    1）list集合 
    
```xml
    <property name="empList">
      <list>
        <ref bean="empone"></ref>
        <ref bean="emptwo"></ref>
      </list>
    </property>
```

​    2）map集合
   ```xml
    <property name="teacherMap">
      <map>
        <entry>
          <key>
            <value>10000</value>
          </key>
          <ref bean="teacherOne"></ref>
        </entry>
        <entry>
          <key>
            <value>10001</value>
          </key>
          <ref bean="teacherTwo"></ref>
        </entry>
      </map>
    </property>
   ```



​    3）引用集合类型的bean

```xml
<util:list id="lessonList">
  <ref bean="lessonOne"></ref>
  <ref bean="lessonTwo"></ref>
</util:list>
<util:map id="teacherMap">
  <entry>
    <key>
      <value>1001</value>
    </key>
    <ref bean="teacherOne"></ref>
  </entry>
  <entry>
    <key>
      <value>1002</value>
    </key>
    <ref bean="teacherTwo"></ref>
  </entry>
</util:map>
```

> 需要在xml中增加相关约束

```xml
...
xmlns:util="http://www.springframework.org/schema/util"
xsi:schemaLocation="http://www.springframework.org/schema/util
                    http://www.springframework.org/schema/util/spring-util.xsd
										..."
```

​	4）p命名空间注入

```xml

<!-- 增加约束 -->
xmlns:p="http://www.springframework.org/schema"
    
```

```xml
<!-- p命名空间注入-->
<bean id="studentP" class="com.example.spring.iocxml.di.Student"
      p:sid="122"
      p:sname="小李"
      p:lessonList-ref="lessonList"
      p:teacherMap-ref="teacherMap"
      ></bean>
```

#### 引入外部文件注入

> 常见场景：数据库配置文件			

1）引入相关依赖

```xml
<!-- 数据库驱动 -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.38</version>
</dependency>
<!-- 数据库连接池 -->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.10</version>
</dependency>
```

2）创建外部属性文件jdbc.properties

```
jdbc.user=root
jdbc.password=123456
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:8306/test?useSSL=false
```

3）创建spring配置文件，引入context命名空间，引入属性文件，使用表达式完成注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd"
>
    <!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
    <!-- 完成注入 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.url}"></property>
        <property name="driverClassName" value="${jdbc.driver}"></property>
        <property name="username" value="${jdbc.user}"></property>
        <property name="password" value="${jdbc.password}"></property>

    </bean>
</beans>
```

4）配置bean

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
  <property name="url" value="${jdbc.url}"></property>
  <property name="driverClassName" value="${jdbc.driver}"></property>
  <property name="username" value="${jdbc.user}"></property>
  <property name="password" value="${jdbc.password}"></property>
</bean>
```

5）测试

```java
@Test
public void testDemo2(){
  ApplicationContext context = new ClassPathXmlApplicationContext("beans-jdbc.xml");
  DruidDataSource dataSource = context.getBean("dataSource",DruidDataSource.class);
  System.out.println(dataSource.getUrl());
}
```

#### bean的作用域

1）**概念**

在Spring中可以通过配置bean标签的scope属性来指定bean的作用域范围

| 取值      | 含义                                    | 创建对象的时机 |
| --------- | --------------------------------------- | -------------- |
| singleton | 在IoC容器中，这个bean的对象始终为单实例 | IoC容器初始化  |
| prototype | 这个bean在IoC容器中有多个实例           | 获取bean时     |

```xml
<!-- 配置单实例-->
<bean id="user" class="com.example.spring.iocxml.di.scope.User" scope="singleton"></bean>
<!-- 配饰多实例 -->
<bean id="userTwo" class="com.example.spring.iocxml.di.scope.User" scope="prototype"></bean>
```

#### bean的生命周期

1）调用无参数构造器，创建bean对象

2）给bean对象设置属性值

3）bean后置处理器（初始化之前）

4）bean对象初始化（调用指定的初始化方法）

5）bean后置处理器（初始化之后）

6）bean对象创建完成了，可以使用了

7）bean对象销毁（配置指定销毁的方法）

8）IoC容器关闭

- 第一步：创建类

  User.java

```java
  package com.example.spring.iocxml.di.lifecycle;
  
  public class User {
      private  String name;
  
      public User(){
          //1.调用无参构造，创建对象
          System.out.println("1. bean对象创建，调用无参数构造器");
      }
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          System.out.println("2. 给bean对象设置属性值");
          this.name = name;
      }
      //初始化方法
      public void initMethod(){
          System.out.println("4.调用指定的初始化方法");
      }
      //销毁方法
      public void destroyMethod(){
          System.out.println("7.调用指定的销毁方法");
      }
      @Override
      public String toString() {
          return "User{" +
                  "name='" + name + '\'' +
                  '}';
      }
  }
  
```

  MyPost.java

```java
  package com.example.spring.iocxml.di.lifecycle;
  import org.springframework.beans.BeansException;
  import org.springframework.beans.factory.config.BeanPostProcessor;
  
  public class MyPost implements BeanPostProcessor {
      @Override
      public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
          System.out.println("5. bean后置处理器（初始化之后）");
          return bean;
      }
      @Override
      public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
          System.out.println("3. bean后置处理器（初始化之前）");
          return bean;
      }
  }
  
```

- 第二步：配置bean

```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="user" class="com.example.spring.iocxml.di.lifecycle.User" init-method="initMethod" destroy-method="destroyMethod">
          <property name="name" value="张三"></property>
      </bean>
      <bean id="myPost" class="com.example.spring.iocxml.di.lifecycle.MyPost"></bean>
  </beans>
```

- 第三步：测试

```java
  @Test
  public void testUserLifeDemo(){
    ClassPathXmlApplicationContext context= new ClassPathXmlApplicationContext("beans-lifecycle.xml");
    User user = context.getBean("user",User.class);
    System.out.println("6.bean对象创建完成，可以使用了");
    System.out.println(user);
    context.close();
  }
```

  

#### FactoryBean

简介：FactoryBean是Spring提供的一种整合第三方框架的常用机制。和普通的bean不同，配置一个FactoryBean类型的bean，在获取bean的时候得到的并不是class属性中配置的这个类的对象，而是getObject()方法的返回值。通过这种机制，Spring可以帮我们把复杂组件创建的详细过程和繁琐细节屏蔽起来，只把最简洁的使用界面展示给我们。

将来我们整合MyBatis时，Spring就是通过FactoryBean机制来帮我们创建SqlSessionFactory对象的。

**1）创建类UserFactoryBean**

```java
package com.example.spring.iocxml.di.factorybean;

import org.springframework.beans.factory.FactoryBean;

public class MyFactoryBean extends User implements FactoryBean<User> {
    @Override
    public User getObject() throws Exception {
        return new User();
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }
}

```

**2）配置bean**

```xml
<bean id="myFactoryBean" class="com.example.spring.iocxml.di.factorybean.MyFactoryBean"></bean>
```

**3）测试**

```java
@Test
public void testMyFactoryBeanDemo(){
  ApplicationContext context = new ClassPathXmlApplicationContext("beans-factorybean.xml");
  User user = (User)context.getBean("myFactoryBean");
  System.out.println(user);
}
```

#### 基于xml自动装配

> 根据指定的策略，在IoC容器中匹配某一个bean，自动为指定的bean中所依赖的类类型或接口类型属性赋值

```xml
<!--
  	自动装载: autowire
               |- byType 根据类型进行注入
               |- byName 根据名称进行注入
-->
<bean id="userController" class="com.example.spring.iocxml.auto.controller.UserController" autowire="byType"></bean>
<bean id="userService" class="com.example.spring.iocxml.auto.service.UserServiceImpl" autowire="byType"></bean>
<bean id="userDao" class="com.example.spring.iocxml.auto.dao.UserDaoImpl"></bean>
```

### 基于注解管理bean

#### 概念

代码中的一种特殊标记，格式：@注解名称(属性1=属性值...)；简化Spring的XML配置；

Spring 通过注解实现自动装配的步骤如下：

1. 引入依赖
2. 开启组件扫描
3. 使用注解定义Bean
4. 依赖注入

##### 引入依赖

```xml
<dependencies>
  <!-- 测试 -->
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
  </dependency>
  <!-- Spring 核心-->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.18</version>
  </dependency>
  <!-- 日志相关 -->
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
  </dependency>
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
  </dependency>
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
  </dependency>
</dependencies>
```

> 注意：logback.xml放在resource下，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!-- 其他配置-->
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%level] %blue(%date{yyyy-MM-dd HH:mm:ss.SSS}) %cyan([%thread]) %boldGreen(%logger{15})- %msg %n</pattern>
        </encoder>
    </appender>
    <!--
        1. 用来指定某一个包或者具体某一个类的日志打印级别，以及指定<appender>
        2. <logger>只有一个name属性，一个可选的level和一个可选的additivity
     -->
    <root level="debug">
        <appender-ref ref="Console"/>
    </root>
    <!-- 下面的两个logger是关键的两个logger -->
    <logger name="org.apache.ibatis" level="DEBUG">
        <appender-ref ref="Console"/>
    </logger>
    <logger name="java.sql" level="debug">
        <appender-ref ref="Console"/>
    </logger>
</configuration>
```

##### 开启组件扫描

新建beans.xml，引入context命名空间，开启组件扫描。开启扫描后，Spring会自动扫描配置路径下的所有java类，并创建Bean实例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.example"></context:component-scan>
</beans>
```



##### 使用注解定义Bean

Spring提供了以下几个注解来标注Spring Bean

@Component：标注一个普通的Spring Bean类

@Controller：标注一个控制器组件类

@Service：标注一个业务逻辑组件类

@Repository：标注一个DAO组件类

```java
@Component(value = "user") //value可以省略，默认就是类名首字母小写
public class User {
}
```

##### 依赖注入

@Autowired 

1) 属性注入

```java
@Controller
public class UserController {
    //第一种方式：属性注入
    @Autowired
    private UserService userService;
    public void out(){
        System.out.println("userController ......");
        userService.out();
    }
}
```

```java
@Repository
public class UserDaoImpl implements UserDao{
    @Override
    public void out() {
        System.out.println("userDao ......");
    }
}
```

```java
@Service
public class UserServiceImpl implements UserService{
    @Autowired
    private UserDao userDao;
    @Override
    public void out() {
        System.out.println("userService ......");
        userDao.out();
    }
}
```

2. setter方法注入

```java
//第二种方式：setter方法注入
private UserService userService;
@Autowired
public void setUserService(UserService userService) {
  this.userService = userService;
}
```

3. 构造方法注入

```java
//第三种方式：构造方法注入
private UserService userService;
@Autowired
public UserController(UserService userService) {
  this.userService = userService;
}
```

4. 形参上注入

```java
//第四种方式：形参注入
private UserService userService;
public UserController(@Autowired UserService userService) {
  this.userService = userService;
}
```

5. @Autowired注解和@Qualifier注解联合实现 按名称自动注入

```java
//第五种方式：@Autowired注解和@Qualifier注解联合实现 按名称自动注入
@Autowired
@Qualifier("userRedisServiceImpl")
private UserService userService;
public void out(){
  System.out.println("userController ......");
  userService.out();
}
```

### 全注解开发

就是不再使用spring配置文件了，写一个配置类来代替配置文件

**配置类**

```java
@Configuration
@ComponentScan("com.example.spring")
public class SpringConfig {
}
```

**测试类**

```java
@Test
public void testDemo(){
  ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
  UserController controller = context.getBean(UserController.class);
  controller.out();
}
```

