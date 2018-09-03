---
title: PostgreSQL使用总结
date: 2018-05-15 19:07:40
tags:
    - 数据库
    - PostgreSQL
categories:
    - 数据库
    - PostgreSQL
---
# PostgreSQL大小写
* PostgreSQL数据库对象名大小写敏感  
对象名：如表名、字段名  
PostgreSQL数据库内核是区分大小写的。  
只是为了方便使用，数据库在分析SQL脚本时，对不加双引号的所有对象名转化为小写字母。  
对象名加上双引号是区分大小写的，不加双引号转化为小写字母。  
例如1：  
`SELECT * FROM "TUser" WHERE "Name" LIKE '%tony%';`  
`SELECT * FROM "tUser" WHERE "name" LIKE '%tony%';`  
两者是不同的，两个的表名、字段名称区分大小写  
例如2：  
`SELECT * FROM TUser WHERE Name LIKE '%tony%';`  
`SELECT * FROM tUser WHERE name LIKE '%tony%';`  
两者是相同的，两个的表名、字段名称会被转换成小写
例1和例2的区别在于是否加双引号  
***注意：表名、字段名只允许双引号，不支持单引号***
* 数据区分大小写  
LIKE '%a%',A是不会出来的  
***注意：数据双引号、单引号都可以，建议使用单引号***

# 清空数据库还原数据库为新建时的状态
在postgresql中，创建数据库时会自动创建public模式，一般我们把表都保存在该模式中，因此直接删除该模式再重新创建该模式。  
若数据在其他模式中，则把public换为数据表所在模式即可。
```
//删除public模式以及模式里面所有的对象
DROP SCHEMA public CASCADE;
//创建public模式
CREATE SCHEMA public;
```
