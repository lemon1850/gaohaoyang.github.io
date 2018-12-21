---
layout: post
title: mybatis那些事
categories: database db orm 框架
tags:  orm mybatis
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境

本文主要mybatis那些事



# 处理in条件

## 问题

写sql经常需要处理到匹配多个条件


```sql
select * from user where id in ('1','2','3')
```

## 处理方案

方法一: 将list转化成字符串


```java
List<Integer> ids = ImmutableList.of(1,2,3);
String idList = "'"  + StringUtils.join(ids, "',") + "'";
```


```xml
 select * from user where id in ( ${idList})
```

方法二: 使用foreach


```xml
select * from user where id in 
<foreach collection="idList" index="index" item="id" open="(" separator="," close=")">
		#{id}
</foreach>
```

# 使用mybatis-generator忽略列


```xml
<table schema="tianhe" tableName="user" domainObjectName="User" mapperName="UserMapper"
               enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false"
               enableSelectByExample="false" selectByExampleQueryId="true" >
    <ignoreColumn column="drc_check_time"></ignoreColumn>
    <ignoreColumn column="ezone_shard_info"></ignoreColumn>
</table>
```



