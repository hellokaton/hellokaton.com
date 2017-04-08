---
layout: post
title: Python操作mysql（增删改查）
categories: python
tags: python mysql
---

```python
#!/usr/bin/env python
#coding:utf-8

import MySQLdb
try:
    #连接mysql的方法：connect('ip','user','password','dbname')    
    #conn=MySQLdb.connect(host='localhost',user='root',passwd='123456',db='test')
    conn =MySQLdb.connect('127.0.0.1','root','123456',charset = 'gb2312')
    conn.select_db('python')
    cur=conn.cursor()
    sql1 = 'drop database python' #删除数据库 
    sql2 = 'create database if not exists python' #若不存在，则创建数据库
    sql3 = 'create database python'
    sql4 = 'create table module(m_id int not null,m_name VARCHAR(25),m_size int)'#创建表
    sql5 = 'create table if not exists demo(d_id int not null,d_name varchar(25),m_size int default 0)'
    values=[]
    for i in range(1): 
        values.append((i,'mysql',i+1)) 
    sql6 = 'insert into module values(%s,%s,%s)'
    #cur.executemany(sql6,values)   #批量插入
    values = [1,'MySQLdb',5]
    sql6 = "insert into module VALUES('%d','%s','%d')"%(2,'MySQLdb',7) #插入
    #sql6 = "insert into module(m_id,m_name,m_size) VALUES('%d','%s','%d')"%(2,'MySQLdb',7)
    #sql6 = "insert into module(m_id,m_name,m_size) VALUES('%d','%s','%d')"%(values[0],values[1],values[2])
    sql7 = "update module set m_name='MySql' where m_id=0 and m_size=0" #修改
    sql8 = "delete from module where m_id=1 and m_size=0" #删除
    sql9 = "select * from module where m_id=1"
    cur.execute(sql9)
    count = cur.execute(sql9)  #查询结果数量
    print u'查询结果数量：',count
    result = cur.fetchone() 
    print u'单条查询结果：',result
    result = cur.fetchmany(2)
    print u'多条查询结果：',result
    result = cur.fetchall()
    print u'所有不同的查询结果：',result
    for data in result:
        print data
    conn.commit()
    cur.close()
    conn.close()
except MySQLdb.Error,e:
    print "Mysql Error %d: %s" % (e.args[0], e.args[1])
```
