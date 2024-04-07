---
title: Mysql常用操作汇总
date: 2024-04-03
categories:
  - 实战
  - Mysql服务器
tags:
  - mysql
  - mysql常用操作
---

# Mysql 常用操作

## 导入

```sh
# 导入一个名为database_name的数据库，该数据库的数据文件是database_name.sql
mysql -u username -p database_name < database_name.sql 
# 在执行上述命令时，系统会提示您输入密码。请将username替换为您的MySQL用户名，如果需要，也可以指定主机和端口：
mysql -h host -P port -u username -p database_name < database_name.sql
# 确保您有足够的权限来导入数据库，并且数据库已经存在于MySQL中，如果不存在，您需要先创建它
mysql -u username -p -e "CREATE DATABASE database_name"
```



# 问题

## timestamp 类型默认‘0000-00-00 00:00:00’ 问题

```sh
 导入报错： Incorrect datetime value: '0000-00-00 00:00:00' for column 'update_time' at row 1
```

### 原因分析：

因为 timestamp 类型的取值范围：1970-01-01 00:00:00 到 2037-12-31 23:59:59

### 处理方法：

设置sql_mode

my.ini

```sh
sql_mode= NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

> 参考链接：https://blog.csdn.net/lxw1844912514/article/details/100028536
>
> 参考链接：https://blog.csdn.net/foreveryangting/article/details/82686081

## Cannot add foreign key constraint

### 原因分析



### 处理方法

