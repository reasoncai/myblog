---
title: mybatis模糊查询like如何写
date: 2017-06-12 12:38:42
tags: mybatis
categories: 
---

##### mysql(利用能连接多个字符串的contat，无||)
```SQL
where username LIKE concat('%',#{username},'%')
where username LIKE concat('%',#{username})
where username LIKE "%"#{username}"%"
```
##### oracle(CONCAT只能连接两个字符串,可以用||)
```SQL
where username LIKE '%' || #{username} || '%' 
where username LIKE concat('%',concat(#{username},'%'))（嵌套concat）
```
##### 两者都支持（不推荐，$导致SQL注入）
```SQL
where username LIKE '%${username}%'
```
ps：当mysql的字段名和保留字冲突的时候如key，sql语句中的字段名需要加上反引号``来加以区别，反引号可以用Esc键下面那个按键在英文模式不按shift键打出来，注意，是反引号不是单引号，回车键左边那个是单引号。