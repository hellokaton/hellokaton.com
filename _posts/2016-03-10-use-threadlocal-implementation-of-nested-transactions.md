---
layout: post
title: 使用ThreadLocal实现Java嵌套事务
categories: java
tags: threadlocal
---

以前编程里大多嵌套事务都是通过 `EJB` 实现的，现在我们尝试实现对POJO的嵌套事务。这里我们使用了ThreadLocal的功能。

<!-- more -->

**理解嵌套事务**

事务是可以嵌套的。所以内层事务或外层事务可以在不影响其他事务的条件下进行回滚或提交。

新建的事务嵌套在外层事务中。如果内层事务完成(不论是回滚或是提交)，外层的事务就可以进行回滚或提交，这样的操作并不会影响内层事务。首先关闭最内层的事务，并逐步移动到外层事务。

![](https://i.imgur.com/A9F7VS2.png)

**使用简单的POJO实现**

新建如下接口：

```java
public interface TransactionManager {
 
    Connection getConnection();
    void beginTransaction();
    void commit();
    void rollback();
}
```

<!--more-->

新建如下事务管理类：

```java
public class TransactionManagerStackImpl implements TransactionManager {
     
    private Stack<Connection>connections = new Stack<Connection>();
 
    @Override
    public Connection getConnection() {
 
        if (connections.isEmpty()) {
            this.addConn();
        }
 
        return connections.peek();
    }
 
    @Override
    public void beginTransaction() {
        this.addConn();
    }
 
    @Override
    public void commit() {
        try {
            if (connections.peek() != null&& !connections.peek().isClosed()) {
                System.out.println(connections.peek().toString() +"--Commit---");
                connections.peek().commit();
                connections.pop().close();
            }
 
        } catch (SQLException e) {
            e.printStackTrace();
        }
 
    }
 
    @Override
    public void rollback() {
        try {
 
            if (connections.peek() != null&& !connections.peek().isClosed()) {
                System.out.println(connections.peek().toString() +"--Rollback---");
                connections.peek().rollback();
                connections.pop().close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
 
    }
 
    private void addConn() {
        try {
            Connection con = this.getMysqlConnection();
            con.setAutoCommit(false);
            connections.push(con);
            System.out.println(con.toString() +"--Conection---");
        } catch (SQLException e) {
            e.printStackTrace();
        }
         
    }
 
    private Connection getMysqlConnection() {
        return getConnection("com.mysql.jdbc.Driver", "jdbc:mysql://localhost:3306/testdb", "test", "test12345");
    }
 
    private Connection getConnection(String driver, String connection,
            String user, String password) {
 
        try {
            Class.forName(driver);
            return DriverManager.getConnection(connection, user, password);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
 
        returnnull;
 
    }
}
```

到这里，我们创建了一个栈(Stack)

```java
private Stack<Connection> connections = new Stack<Connection>();
```

事务遵循栈“先进后出”的原则，通过栈存储事务的连接：

```java
public void beginTransaction()
```

beginTransaction()用于开启一个新的事务，并将连接加入到栈中。自动提交设置为否：

```java
public Connection getConnection()
```

getConnection()获得当前事务的连接。如果连接为空，则创建新的连接并将其加入到栈：

```java
public void commit()
```

提交当前的事务，之后关闭连接，并将其从栈中移除：

```java
public void rollback()
```

回滚当前的事务，之后关闭连接，并将其从栈中移除。

上面的TransactionManagerStackImpl类为单线程创建了嵌套事务。

**多线程的嵌套事务**

在多线程的应用中，每个线程都有其独立的事务和嵌套事务。

我们使用ThreadLocal管理栈的连接。

```java
public class TransactionManagerThreadLocal implements TransactionManager {
     
    private static final ThreadLocal<TransactionManager>tranManager = newThreadLocal<TransactionManager>() {
         
    protected TransactionManager initialValue() {
        System.out.println(this.toString() + "--Thread Local Initialize--");
    return new TransactionManagerStackImpl();
        }
      };
 
    @Override
    public void beginTransaction() {
        tranManager.get().beginTransaction();
    }
 
    @Override
    public void commit() {
        tranManager.get().commit();
    }
 
    @Override
    public void rollback() {
        tranManager.get().rollback();
    }
 
    @Override
    public Connection getConnection() {
        returntranManager.get().getConnection();
    }
}
```

这里初始化TransactionManagerStackImpl，在线程中创建嵌套的事务。

**测试**

测试上面的方法，提交内层事务，回滚外层事务。

```java
public class NestedMain implements Runnable {
     
    private int v = 0;
    private String name;
     
    NestedMain(int v, String name) {
        this.v = v;
        this.name = name;
    }
 
    public static void main(String[] args) throws Exception{
         
        for (inti = 0; i< 3; i++) {
            NestedMain main = newNestedMain(i * 10, "Ravi" + i);
            new Thread(main).start();
        }
    }
 
    @Override
    public void run() {
         
        try {
            TransactionManagerThreadLocal local = new TransactionManagerThreadLocal();
             
            // Transaction 1 ( outer )
            local.beginTransaction();
            Connection con = local.getConnection();
            String sql = "INSERT INTO test_tran (emp_id, name) VALUES ('1"+v+"', '"+ name+v+"')";
            this.insert(con, sql);
     
                // Transaction 2 ( Inner )
                local.beginTransaction();
                con = local.getConnection();
                sql = "INSERT INTO test_tran (emp_id, name) VALUES ('2"+v+"', '"+ name+v+"')";
                this.insert(con, sql);
                local.commit(); // Committing 2
 
            local.rollback(); // Rollback 1 Outer
 
        } catch (Exception e) {
            e.printStackTrace();
        }
	}
}
```

**结果**

```sh
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.mysql.jdbc.JDBC4Connection@10dd1f7--Conection---
com.mysql.jdbc.JDBC4Connection@1813fac--Conection---
com.mysql.jdbc.JDBC4Connection@136228--Conection---
com.mysql.jdbc.JDBC4Connection@1855af5--Conection---
com.mysql.jdbc.JDBC4Connection@e39a3e--Conection---
com.mysql.jdbc.JDBC4Connection@1855af5--Commit---
com.mysql.jdbc.JDBC4Connection@e39a3e--Commit---
com.mysql.jdbc.JDBC4Connection@9fbe93--Conection---
com.mysql.jdbc.JDBC4Connection@9fbe93--Commit---
com.mysql.jdbc.JDBC4Connection@10dd1f7--Rollback---
com.mysql.jdbc.JDBC4Connection@1813fac--Rollback---
com.mysql.jdbc.JDBC4Connection@136228--Rollback---
 
|  name         | emp_id           
| ------------- |:-------------:
| Ravi220       | 220
| Ravi00        | 20      
|Ravi110 	    | 210      
```

内层事务回滚，外层事务提交的情况：

```sh
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.ttit.TransactionManagerThreadLocal$1@1270b73--Thread Local Initialize--
com.mysql.jdbc.JDBC4Connection@9f2a0b--Conection---
com.mysql.jdbc.JDBC4Connection@136228--Conection---
com.mysql.jdbc.JDBC4Connection@1c672d0--Conection---
com.mysql.jdbc.JDBC4Connection@9fbe93--Conection---
com.mysql.jdbc.JDBC4Connection@1858610--Conection---
com.mysql.jdbc.JDBC4Connection@9fbe93--Rollback---
com.mysql.jdbc.JDBC4Connection@1858610--Rollback---
com.mysql.jdbc.JDBC4Connection@1a5ab41--Conection---
com.mysql.jdbc.JDBC4Connection@1a5ab41--Rollback---
com.mysql.jdbc.JDBC4Connection@9f2a0b--Commit---
com.mysql.jdbc.JDBC4Connection@136228--Commit---
com.mysql.jdbc.JDBC4Connection@1c672d0--Commit---
...
|  name         | emp_id           
| ------------- |:-------------:
| Ravi00        | 10
| Ravi220       | 120     
|Ravi110 		| 110
```

原文链接： [javacodegeeks](http://www.javacodegeeks.com/2013/12/java-nested-transaction-using-threadlocal-in-pojo.html)
