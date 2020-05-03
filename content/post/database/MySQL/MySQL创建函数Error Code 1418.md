---
title: "MySQL Error Code 1418"
date: 2019-05-29T19:06:12+08:00
categories:
- Database
- MySQL
tags:
- mysql
- Error Code
- 1418
keywords:
- mysql
- 1418
---

mysql开启bin-log时，创建子程序(存储过程，函数，触发器)会报错`Error Code : 1418`。

<!--more-->

```plaintext
Error Code: 1418. This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)
```

## 原因分析

因为`CREATE PROCEDURE`, `CREATE FUNCTION`, `ALTER PROCEDURE`,`ALTER FUNCTION`, `DROP PROCEDURE`, `DROP FUNCTION`等语句都会被写进二进制日志,然后在从服务器上执行。

但是，一个执行更新操作的不确定子程序(存储过程、函数、触发器)是不可重复的(即不应该在Master节点上执行一次，又在Slaver节点上执行一次)。

为了解决这个问题，MySQL强制要求：

在主服务器上，除非子程序被声明为确定性的或者不更改数据，否则创建或者替换子程序将被拒绝。

这意味着当创建一个子程序的时候，必须要么声明它是确定的，要么它不改变数据。

### 声名确定性

* DETERMINISTIC		&emsp;&emsp;&emsp;&emsp;										确定的
* NOT DETERMINISTIC	&emsp;&emsp;													不确定的

### 声明是否修改数据

* NO SQL			&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;			没有SQl语句，不会修改数据
* READS SQL DATA	&emsp;&emsp;&emsp;&emsp;										只是读取数据，不会修改数据
* MODIFIES SQL DATA	&emsp;&emsp;													要修改数据
* CONTAINS SQL		&emsp;&emsp;&emsp;&emsp;&emsp;									包含了SQL语句

## 解决方案

解决方案有两种：

### 声明确认性或者是否修改数据

在创建子程序时，声明子程序为`DETERMINISTIC` 或 `NO SQL` 或 `READS SQL DATA`的。

```sql
DROP FUNCTION IF EXISTS UnderscoreToCamelCase;

DELIMITER $$
CREATE FUNCTION UnderscoreToCamelCase(content VARCHAR(100)) RETURNS VARCHAR(100) READS SQL DATA 
BEGIN
DECLARE POSITION_UNDER INT DEFAULT 0;
DECLARE V VARCHAR(100) DEFAULT '';
SELECT POSITION('_' in content) INTO POSITION_UNDER;
WHILE POSITION_UNDER>0 DO 
	SELECT concat(V,upper(SUBSTR(content,1,1)),SUBSTR(content,2,POSITION_UNDER-2)) into V;
    SELECT SUBSTR(content,POSITION_UNDER+1,length(content)) into content;
    SELECT POSITION('_' in content) INTO POSITION_UNDER;
END WHILE ; 
SELECT concat(V,upper(SUBSTR(content,1,1)),SUBSTR(content,2,length(content))) into V;
RETURN V ;
END
$$
DELIMITER ;
```
上面的代码中，`CREATE FUNCTION ` 行尾的`READS SQL DATA `声明这个函数是只是读取数据的。


### 信任函数创建者


* 在客户端上执行`SET GLOBAL log_bin_trust_function_creators=1`;
* MySQL服务器启动命令加上`--log-bin-trust-function-creators`选项，参数设置为1
* 在MySQL配置文件`my.ini`或`my.cnf`中的`[mysqld]`段上加`log-bin-trust-function-creators=1`
