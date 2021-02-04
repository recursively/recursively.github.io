---
title: Secure Coding Practice - Injection
date: 2020-10-15 09:16:20
categories: SCP
tags: [Injection, SQL injection, SCP]
keywords: [Injection, SQL injection, SCP]
description: This series is about secure coding during development. The OWASP TOP 10 is our guide to define vulnerability type. For this part, it will focus on the vulnerable points and the mitigation of injection problems.
---
## Overview
Injection flaws, such as SQL, NoSQL, OS, and LDAP injection, occur when untrusted data is sent to an interpreter as part of a command or query. The attacker’s hostile data can trick the interpreter into executing unintended commands or accessing data without proper authorization.

## SQL injection
The most common example acount the injection problem is SQL injection. Here we're gonna discuss the vulnerable code and mitigation for different coding languages.

### Go
The input from users can be arbitrary characters, if you have a query such as the one below:
```Go
key := r.URL.Query().Get("username")
rows, err := db.Query("SELECT username, password FROM accounts WHERE username = '" + key + "'")
```
When provided a valid username value, the program will only list the matched user infomation. But when the input is 1' or '1'='1, the query will be like:
```
SELECT username, password FROM accounts WHERE username = '1' OR '1'='1'
```
All the table records will be dumped with this statement because '1'='1' will be always true. Even worse, if the program has the permission for writing, the whole backend system is about to be exploited.

Prepared statements is the one way to keep the database safe:
```Go
key := r.URL.Query().Get("username")
rows, err := db.Query("SELECT username, password FROM accounts WHERE username = ?", key)
```

### Java
The common vulnerable implemention to query the database in Java:
```Java
String query = "SELECT username, password FROM accounts WHERE username = '" + key + "'"
stm = con.createStatement();
rs = stm.executeQuery(query);
```
The whole database is vulnerable to the malicious input value. The proposal is to use prepared statement to protect database from arbitrary input.
```Java
PreparedStatement ps = null;
String query = "SELECT username, password FROM accounts WHERE username = ?";
ps = con.prepareStatement(query);
ps.setString(1, customerId);
rs = ps.executeQuery();
```

### MariaDB
In fact the prepared statement is the feature of database in order to improve Data retrieval efficiency, this feature can also protect database from injection attack. That means you can do prepared statements without any special API support from your programming language. If the native API is not available, the operation below can achieve the same goal.
```Shell
MariaDB [security]> prepare stmt from 'SELECT password FROM accounts WHERE username = ?';
Query OK, 0 rows affected (0.008 sec)
Statement prepared
 
MariaDB [security]> set @param = "tester1";
Query OK, 0 rows affected (0.000 sec)
 
MariaDB [security]> execute stmt using @param;
+----------+
| password |
+----------+
| 654321   |
+----------+
1 row in set (0.001 sec)
```
All the query statement will be pre-analysed except the placeholder "?". Placeholder syntax in prepared statements is database-specific. For example, comparing MySQL, PostgreSQL, and Oracle:

| MySQL           | PostgreSQL         | Oracle                      |
| :-------------: | :----------------: | :-------------------------: |
| WHERE col = ?   | WHERE col = $1     | WHERE col = :col            |
| VALUES(?, ?, ?) | VALUES($1, $2, $3) | VALUES(:val1, :val2, :val3) |

## Mitigation
There are three primary defenses to avoid SQL injection:

* Use of Prepared Statements 
* Use of Stored Procedures(Only if stored procedure does not generate dynamic SQL)
* Whitelist Input Validation

## Reference
https://owasp.org/www-project-top-ten/2017/Top_10.html
