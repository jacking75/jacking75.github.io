---
layout: post
title: MySQL 8.0.16의 CHECK 제약
published: true
categories: [DB]
tags: db mysql mysql8
---
[참고](https://tech-lab.sios.jp/archives/18478 )  
  
CHECK 제약 조건을 지정하여 테이블을 만든다. (※ t1.a에 "0 < a AND a <= 3" 이라는 CHEKC 제약을 지정한다)  
<pre>
mysql> SELECT version();
+-----------+
| version() |
+-----------+
| 8.0.16    |
+-----------+
1 row in set (0.00 sec)

mysql> 
mysql> CREATE TABLE t1 (a INT CHECK (0 < a AND a <= 3));
Query OK, 0 rows affected (0.10 sec)

mysql> 
</pre>  
  
  
테이블 작성 후, SHOW CREATE TABLE 명령에서 테이블 정의를 확인하여 보면 CHECK 제약 조건 설정 `CONSTRAINT t1_chk_1CHECK (((0 < a) and ( a<= 3)))` 가 반영 되어 있다.  
<pre>
mysql> SHOW CREATE TABLE t1;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                  |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `a` int(11) DEFAULT NULL,
  CONSTRAINT `t1_chk_1` CHECK (((0 < `a`) and (`a` <= 3)))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> 
</pre>  
   
   
   
아까와 마찬가지로이 테이블 t1에 대해 실제로 값을 INSERT 해 본다. 우선 CHECK 제약 조건에 지정한 조건 `0 <a AND a <= 3`의 범위 안의 값을 INSERT 해 본다.  
<pre>
mysql> INSERT INTO t1 VALUES (1), (2), (3);
Query OK, 3 rows affected (0.05 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> 
mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.01 sec)

mysql> 
</pre>  
  
별 문제없이 INSERT 되었다. 다음으로 CHECK 제약 조건에 지정한 조건 `0 <a AND a <= 3`의 범위를 벗어나는 값을 INSERT 해 본다.  
<pre>  
mysql> INSERT INTO t1 VALUES (0);
ERROR 3819 (HY000): Check constraint 't1_chk_1' is violated.
mysql> 
mysql> INSERT INTO t1 VALUES (4);
ERROR 3819 (HY000): Check constraint 't1_chk_1' is violated.
mysql> 
mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

mysql> 
</pre>  
  
이번에는 앞과 달리 "Check constraint 't1_chk_1'is violated." 라는 오류 메시지가 출력 되었다.  
실제로 t1의 기록을 확인하면 오류가 되어 "0" 또는 "4"의 값은 INSERT 되지 않은 것을 확인할 수 있다.  
MySQL 8.0.16에서 CHECK 제약 조건이 구현 되어 있는 것을 확인할 수 있었다.
