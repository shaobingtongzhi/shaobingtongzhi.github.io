---
title: MyBatis学习笔记
date: 2023-04-28 19:19:48
tags: 
    - Spring
    - MyBatis
categories:
    - 学习笔记
    - 02-MyBatis学习笔记
---

# MyBatis 用法

[MyBatis文档](https://mybatis.org/mybatis-3/zh/index.html)

## MyBatis入门
### 查询user表中所有数据
1. 创建user表，添加数据
2. 创建模块，导入坐标
3. 编写 MyBatis 核心配置文件 --> 替换连接信息，解决硬编码问题  (mybatis-config.xml放在资源目录下)
4. 编写 SQL 映射文件 --> 统一管理sql语句，解决硬编码问题 (如UserMapper.xml等，放在资源目录下对应的包目录下，包目录与实体类目录结构保持一致)
5. 编码
   1）定义POJO类(实体类)
   2）加载核心配置文件，获取 SqlSessionFactory 对象
   3）获取SqlSession 对象，执行 SQL 语句
   4）释放资源
## Mapper代理开发

> 它不依赖于字符串字面值，会更安全一点

**示例**
```java
sqlSession.getMapper("UserMapper.class")
userMapper.selectAll();
```
1. 定义与SQL映射文件同名的Mapper接口，并且将Mapper接口和SQL映射文件放置在同一目录下
2. 设置SQL映射文件的namespace属性为Mapper接口全限定名
3. 在Mapper接口中定义方法，方法名就是SQL映射文件中的SQL语句的id，并保持参数类型和返回值类型一致
4. 编码 
    1）通过SqlSession的getMapper方法获取Mapper接口的代理对象
    2）调用对应方法完成sql的执行

**注意：**如果Mapper接口名称和SQL映射文件名称相同，并在同一目录下，则可以使用包扫描的方式简化SQL映射文件的加载
mybatis-config.xml
```xml
<mappers>
    <!-- <mapper resource="UserMapper.xml"/> -->
    <package name="com.example.mapper" />
</mappers>
```

## MyBatis核型配置文件
- environments 可以配置成适应多种环境,不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。
- typeAliases 类型别名：它仅用于 XML 配置，意在降低冗余的全限定类名书写

## 配置文件完成增删改查
> 安装MyBatisX插件

1. 查询
    - 查询所有数据
  
步骤：
a)编写mapper接口
b)编写sql语句
c)执行测试方法


    - 查询详情
    - 条件查询
        1）多条件查询
        <if test=""></test>
        2）单条件查询
        <choose>
            <when test=""></when>
            <otherwise></otherwise>
        </choose>


2. 添加
获取插入数据的ID   useGeneratedKeys、keyProperty  
```xml
<insert id="add" useGeneratedKeys="true" keyProperty="id">
    insert into tb_brand (brand_name, company_name, orderd, description, status)
    values (#{brandName},#{companyName},#{orderd},#{description},#{status});
</insert>
```

3. 修改
- 修改全部字段
- 修改动态字段
```xml
<update id="update">
    update tb_brand
    <set>
        <if test="brandName != null and brandName != ''">
            brand_name = #{brandName},
        </if>
        <if test="companyName != null and companyName != ''">
            company_name = #{companyName},
        </if>
        <if test="ordered != null">
            orderd = #{ordered},
        </if>
        <if test="description != null and description != ''">
            description = #{description},
        </if>
        <if test="status != null">
            status = #{status}
        </if>
    </set>
    where id = #{id}
    ;
</update>
```

4. 删除
- 删除一个

- 批量删除
```xml
<delete id="deleteByIds">
    delete from tb_brand where id in
        <foreach collection="ids" open="(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    ;
</delete>
```

## 注解完成增删改查
@Select、@Insert、@Delete、@Update


## 实战入门

### 引入相关依赖

```xml
<!-- MyBatis核心 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.5.5</version>
</dependency>
<!-- mysql驱动 -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.46</version>
</dependency>
```

### 配置文件

在resources目录下新建mybatis-config.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 别名 -->
    <typeAliases>
        <package name="com.example.pojo"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:8306/test?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- 加载SQL的映射文件 -->
        <!-- <mapper resource="com/example/mapper/UserMapper.xml"/> -->
        <!-- Mapper代理的开发方式 -->
        <package name="com.example.mapper"/>
    </mappers>
</configuration>
```

### 新建POJO类

全路径：src/main/java/com/example/pojo/Brand.java

```java
public class Brand {
    private Integer id;
    private String brandName;
    private String companyName;
    private Integer ordered;
    private String description;
    private Integer status;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getBrandName() {
        return brandName;
    }

    public void setBrandName(String brandName) {
        this.brandName = brandName;
    }

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public Integer getOrdered() {
        return ordered;
    }

    public void setOrdered(Integer ordered) {
        this.ordered = ordered;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    @Override
    public String toString() {
        return "Brand{" +
                "id=" + id +
                ", brandName='" + brandName + '\'' +
                ", companyName='" + companyName + '\'' +
                ", ordered=" + ordered +
                ", description='" + description + '\'' +
                ", status=" + status +
                '}';
    }
}
```



### 安装 MyBatisX 插件

打开IDEA->Preferences->插件，安装MyBatisX

作用：

### 创建mapper接口

全路径：src/main/java/com/example/mapper/BrandMapper.java

```java
public interface BrandMapper {
    /*
    * 查询所有
    * */
    List<Brand> selectAll();
    Brand selectById(int id);
    List<Brand> selectListOne(int id);
    /* 多条件查询*/
    //1.散装参数
    //List<Brand> selectByCondition(@Param("status") int status, @Param("brandName") String brandName, @Param("companyName") String companyName);
    //2.对象参数
    //List<Brand> selectByCondition(Brand brand);
    //3.集合参数
    List<Brand> selectByCondition(Map map);
    /* 单条件动态条件查询 */
    List<Brand> selectByConditionSingle(Brand brand);
    /*
    * 添加
    * */
    void add(Brand brand);

    /*
    * 修改
    * */
    int update(Brand brand);

    /*
    * 删除
    * */
    void deleteById(int id);
    /*
    * 批量删除
    * */
    void deleteByIds(@Param("ids") int[] ids);
}

```

### 创建对应的mapper映射文件

全路径：src/main/resources/com/example/mapper/BrandMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
namespace 名称空间
-->
<mapper namespace="com.example.mapper.BrandMapper">
    <!--
        数据库表的 字段名称 和 实体类的 属性名称不一致，则不能自动封装数据
        解决方案：
        1. 起别名，让数据查询的字段名称与实体类属性名称一致
            缺点：每次查询都要定义别名
                解决办法：定义sql片段
                    缺点：不灵活
        2. resultMap
    -->
    <!-- 方案1 起别名-->
    <!-- sql片段 -->
    <!--
    <sql id="brand_column">id, brand_name brandName, company_name companyName, orderd, description, status</sql>
    <select id="selectAll" resultType="brand">
        select
            <include refid="brand_column" />
        from tb_brand;
    </select>
    -->

    <!-- 方案2 resultMap -->
    <resultMap id="brandResultMap" type="brand">
        <result property="brandName" column="brand_name" />
        <result property="companyName" column="company_name" />
    </resultMap>


    <select id="selectAll" resultMap="brandResultMap">
        select *
        from tb_brand;
    </select>
    <select id="selectById" resultMap="brandResultMap">
        select *
        from tb_brand where id = #{id};
    </select>
    <!--
        特殊字符处理：< == &lt;
            方案1： 转义
            方案2：<![CDATA[]]>

    -->
    <!-- 方案1 -->
    <!--<select id="selectListOne" resultMap="brandResultMap">
        select *
        from tb_brand where id &lt; #{id};
    </select>-->
    <!-- 方案2 -->
    <select id="selectListOne" resultMap="brandResultMap">
        select *
        from tb_brand where id
            <![CDATA[
                <
            ]]>
        #{id};
    </select>
    <!-- 多条件查询1 -->
    <!--<select id="selectByCondition" resultMap="brandResultMap">
        select *
        from tb_brand
        where
        status = #{status}
        and brand_name like #{brandName}
        and company_name like #{companyName};
    </select>-->
    <!-- 多条件查询2 -->
    <select id="selectByCondition" resultMap="brandResultMap">
        select *
        from tb_brand
        <where>
            <if test="status != null ">
                status = #{status}
            </if>
          <if test="brandName != null and brandName != ''">
              and brand_name like #{brandName}
          </if>
          <if test="companyName != null and companyName != ''">
            and company_name like #{companyName};
          </if>
        </where>
    </select>


    <!-- 单条件动态条件查询 -->
    <select id="selectByConditionSingle" resultMap="brandResultMap">
        select *
        from tb_brand
        <where>
            <choose>
                <when test="status != null">
                    status = #{status}
                </when>
                <when test="brandName != null and brandName != ''">
                    and brand_name like #{brandName}
                </when>
                <when test="companyName != null and companyName != ''">
                    company_name like #{companyName}
                </when>
                <otherwise></otherwise>
            </choose>
        </where>
        ;
    </select>

    <!-- 添加方法 -->
    <insert id="add" useGeneratedKeys="true" keyProperty="id">
        insert into tb_brand (brand_name, company_name, orderd, description, status)
        values (#{brandName},#{companyName},#{orderd},#{description},#{status});
    </insert>

    <!-- 修改 -->
    <update id="update">
        update tb_brand
        <set>
            <if test="brandName != null and brandName != ''">
                brand_name = #{brandName},
            </if>
            <if test="companyName != null and companyName != ''">
                company_name = #{companyName},
            </if>
            <if test="ordered != null">
                orderd = #{ordered},
            </if>
            <if test="description != null and description != ''">
                description = #{description},
            </if>
            <if test="status != null">
                status = #{status}
            </if>
        </set>
        where id = #{id}
        ;
    </update>

    <delete id="deleteById">
        delete
        from tb_brand
        where
        id = #{id};
    </delete>
    <delete id="deleteByIds">
        delete from tb_brand where id in
            <foreach collection="ids" open="(" close=")" item="id" separator=",">
                #{id}
            </foreach>
        ;
    </delete>
</mapper>
```

### 测试

```java
public class MyBatisTest {
    @Test
    public void testSelectAll() throws IOException {

        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 执行sql
        List<Brand> brands = mapper.selectAll();
        System.out.println(brands);
        //5. 释放资源
        sqlSession.close();
    }

    @Test
    public void testSelectById() throws IOException {
        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 执行sql
        Brand brand = mapper.selectById(2);
        System.out.println(brand);
        //5. 释放资源
        sqlSession.close();
    }
    @Test
    public void testSelectList() throws IOException {
        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 执行sql
        List<Brand> brands = mapper.selectListOne(2);
        System.out.println(brands);
        //5. 释放资源
        sqlSession.close();
    }

    @Test
    public void testSelectByCondition() throws IOException {
        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 执行sql
        int status = 1;
        String brandName = "鸿蒙";
        String companyName = "华为";

        //参数处理
        brandName = "%"+ brandName +"%";
        companyName = "%" + companyName + "%";
        //1.散装参数传递
        //List<Brand> brands = mapper.selectByCondition(status,brandName,companyName);
        //2.对象参数传递
        //Brand brand = new Brand();
        //brand.setStatus(status);
        //brand.setBrandName(brandName);
        //brand.setCompanyName(companyName);
        //List<Brand> brands = mapper.selectByCondition(brand);
        //3.map参数传递
        HashMap hashMap = new HashMap();
        hashMap.put("status",status);
        hashMap.put("companyName",companyName);
        hashMap.put("brandName",brandName);

        List<Brand> brands = mapper.selectByCondition(hashMap);
        System.out.println(brands);
        //5. 释放资源
        sqlSession.close();
    }
    @Test
    public void testSelectByConditionSingle() throws IOException {
        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 执行sql
        int status = 1;
        String brandName = "鸿蒙";
        String companyName = "华为";

        //参数处理
        brandName = "%"+ brandName +"%";
        companyName = "%" + companyName + "%";
        //1.散装参数传递
        //List<Brand> brands = mapper.selectByCondition(status,brandName,companyName);
        //2.对象参数传递
        Brand brand = new Brand();
//        brand.setStatus(status);
        brand.setBrandName(brandName);
        //brand.setCompanyName(companyName);
        List<Brand> brands = mapper.selectByConditionSingle(brand);
        System.out.println(brands);
        //5. 释放资源
        sqlSession.close();
    }

    @Test
    public void testAdd() throws IOException {
        //初始数据
        int status = 1;
        String brandName = "三星2";
        String companyName = "三星技术股份有限公司2";
        int ordered = 2;
        String description = "是韩国最大的跨国企业集团，三星集团包括众多的国际下属企业";

        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 数据准备
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setBrandName(brandName);
        brand.setCompanyName(companyName);
        brand.setOrdered(ordered);
        brand.setDescription(description);
        //5.执行sql
        mapper.add(brand);

        int id = brand.getId();
        System.out.println(id);
        //6. 释放资源
        sqlSession.close();
    }

    @Test
    public void testUpdate() throws IOException {
        //初始数据
        int status = 1;
        String brandName = "三星xxx";
        String companyName = "三星技术股份有限公司xxx";
        int ordered = 2;
        String description = "是韩国最大的跨国企业集团，三星集团包括众多的国际下属企业xxxx";

        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 数据准备
        Brand brand = new Brand();
        brand.setStatus(status);
        brand.setBrandName(brandName);
        brand.setCompanyName(companyName);
        brand.setOrdered(ordered);
        brand.setDescription(description);
        brand.setId(7);
        //5.执行sql
        int res = mapper.update(brand);

        System.out.println(res);
        //6. 释放资源
        sqlSession.close();
    }

    @Test
    public void testDeleteById() throws IOException {
        //初始数据
        int id = 6;

        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 数据准备

        //5.执行sql
        mapper.deleteById(id);

        //6. 释放资源
        sqlSession.close();
    }
    @Test
    public void testDeleteByIds() throws IOException {
        //初始数据
        int[] ids = {4,7};

        //1. 加载配置文件，获取SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        //2. 获取SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //3. 获取Mapper对象
        BrandMapper mapper = sqlSession.getMapper(BrandMapper.class);
        //4. 数据准备

        //5.执行sql
        mapper.deleteByIds(ids);

        //6. 释放资源
        sqlSession.close();
    }
}
```

