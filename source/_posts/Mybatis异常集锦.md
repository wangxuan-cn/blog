---
title: Mybatis异常集锦
date: 2018-04-24 11:37:07
tags:
    - MyBatis
categories:
    - MyBatis
---
# 1. 判断非空的坑
Mybatis中，alarmType是int类型。如果alarmType为0的话，条件判断返回结果为false，其它值的话，返回true。
```
<if test="alarmType != null and alarmType != ''">
   alarm_type=#{alarmType},
</if>
```
其实对于条件判断alarmType如果为0，条件判断结果为true  
`<if test="alarmType == ''">`  
其实如果alarmType 是int类型的话，不用进行非空判断。

**注意：判空操作是对字符串而言，不要对非字符串判空操作；其中int、long类型，条件判断alarmType如果为0，条件判断结果为true**
