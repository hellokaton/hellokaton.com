---
layout: post
title: mysql按日期分组
categories: mysql
tags: mysql
---

```sql
select month(FROM_UNIXTIME(time)) from table_name group by month(FROM_UNIXTIME(time))
 
1、按月分组：
select month(FROM_UNIXTIME(time)) from table_name group by month(FROM_UNIXTIME(time))
2、按年月分组：
select DATE_FORMAT(FROM_UNIXTIME(time),"%Y-%m") from tcm_fund_list group by DATE_FORMAT(FROM_UNIXTIME(time),"%Y-%m")
或者：
select FROM_UNIXTIME(time,"%Y-%m") from tcm_fund_list group by FROM_UNIXTIME(time,"%Y-%m")
3、按年月日分组：
select DATE_FORMAT(FROM_UNIXTIME(time),"%Y-%m-%d") from tcm_fund_list group by DATE_FORMAT(FROM_UNIXTIME(time),"%Y-%m-%d")
或者：
select FROM_UNIXTIME(time,"%Y-%m-%d") from tcm_fund_list group by FROM_UNIXTIME(time,"%Y-%m-%d")
其中FROM_UNIXTIME(time,"%Y-%m-%d")中的time代表UNIX时间戳的字段名称
```