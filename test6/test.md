# 基于Oracle的商城订单的数据库设计
## 一、引言
### 1.1编写目的
本数据库设计说明书是对商城订单数据库设计定义，包括本系统数据库设计、数据库的实现代码、在数据库中建立一个程序包，在包中用PL/SQL语言设计一些存储过程和函数，实现比较复杂的业务逻辑，用模拟数据进行执行计划分析，以及设计数据库备份方案和设计容灾方案。
### 1.2背景
数据库(Database) 是按照数据结构来组织、存储和管理数据的仓库，它产生于距今五十年前,随着信息技术和市场的发展,特别是二十世纪九十年代以后，数据管理不再仅仅是存储和管理数据,而转变成用户所需要的各种数据管理的方式。数据库有很多种类型，从最简单的存储有各种数据的表格到能够进行海量数据存储的大型数据库系统都在各个方面得到了广泛的应用。
数据库的种类很多，常用的有DB2, Oracle, Sybase, SQL Server, mySQL, Access等，各种数据库各有优劣。其中Oracle以其使用方便，安全性高，功能强大和完善的企业服务系统赢得了众多用户的青睐。
在人类社会已经迈入21世纪的今天，人们深刻的感受到了Oracle在生活和工作中的作用越来越重要,越来越多的职业需要具有Oracle的应用技能。今天，计算机技术不但广泛地应用在办公自动化中，还全面渗透到各行各业。所有与相关的职业都要求工作者有很强的计算机操作技能，做到运用自如，熟练而且深入的掌握软件的应用。随着计算机技术的飞速发展，今后，计算机作为一种崭新的生产力，将在信息社会即新技术革命中发挥越来越重要的作用，并进一步推动人类社会更快的向前发展。其中Oracle数据库起着重要的作用。
如今企业面临着越来越激烈的外部竞争环境,考传统的经营方式已很难适应这种变化，企业现在需要的是先进的经营管理理念和高效的资源管理方式。这就需要引进高效，合理，针对性强的数据库管理系统，而oracle则是世界上大多数企业的首选，可以看出oracle在企业信息化建设中起到的巨大作用。
## 二、数据库设计
### 2.1创建表空间
（1）创建表空间user10
运行代码如下：

```sql
Create Tablespace user10                 
datafile
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user10_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED,
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user10_2.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126210830486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



（2）创建表空间user22
运行代码如下：

```sql
Create Tablespace user22
datafile
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user22_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED,
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user22_2.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112621085812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




### 2.2创建角色和用户，并授权和分配空间
(1)创建角色jh1和用户jh
运行代码如下：

```sql
CREATE ROLE jh1;
 GRANT connect,resource,CREATE VIEW TO jh1;
CREATE USER jh IDENTIFIED BY 123 DEFAULT TABLESPACE user10 TEMPORARY TABLESPACE temp;
 ALTER USER jh QUOTA 50M ON users;
 GRANT jh1 TO jh;
 exit
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211031911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




(2)创建角色jh2和用户jh3
运行代码如下：

```sql
SQL> CREATE ROLE jh2;
SQL> GRANT connect,resource,CREATE VIEW TO jh2;
SQL> CREATE USER jh3 IDENTIFIED BY 123 DEFAULT TABLESPACE user22 TEMPORARY TABLESPACE temp;
SQL> ALTER USER jh3 QUOTA 50M ON users;
SQL> GRANT jh2 TO jh3;
SQL> exit
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211050339.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



### 2.3创建5张表
（1）创建DEPARTMENTS表
运行代码如下：

```sql
CREATE TABLE DEPARTMENTS
(
  DEPARTMENT_ID NUMBER(6, 0) NOT NULL
, DEPARTMENT_NAME VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT DEPARTMENTS_PK PRIMARY KEY
  (
    DEPARTMENT_ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX DEPARTMENTS_PK ON DEPARTMENTS (DEPARTMENT_ID ASC)
      NOLOGGING
      TABLESPACE USER10
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        INITIAL 65536
        NEXT 1048576
        MINEXTENTS 1
        MAXEXTENTS UNLIMITED
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
NOLOGGING
TABLESPACE USER10
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOCOMPRESS NO INMEMORY NOPARALLEL;
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211146190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



（2）创建EMPLOYEES表
运行代码如下：

```sql
CREATE TABLE EMPLOYEES
(
  EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, NAME VARCHAR2(40 BYTE) NOT NULL
, EMAIL VARCHAR2(40 BYTE)
, PHONE_NUMBER VARCHAR2(40 BYTE)
, HIRE_DATE DATE NOT NULL
, SALARY NUMBER(8, 2)
, MANAGER_ID NUMBER(6, 0)
, DEPARTMENT_ID NUMBER(6, 0)
, PHOTO BLOB
, CONSTRAINT EMPLOYEES_PK PRIMARY KEY
  (
    EMPLOYEE_ID
  )
  USING INDEX
  (
      CREATE UNIQUE INDEX EMPLOYEES_PK ON EMPLOYEES (EMPLOYEE_ID ASC)
      NOLOGGING
      TABLESPACE USER10
      PCTFREE 10
      INITRANS 2
      STORAGE
      (
        INITIAL 65536
        NEXT 1048576
        MINEXTENTS 1
        MAXEXTENTS UNLIMITED
        BUFFER_POOL DEFAULT
      )
      NOPARALLEL
  )
  ENABLE
)
NOLOGGING
TABLESPACE USER10
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS UNLIMITED
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NO INMEMORY
NOPARALLEL
LOB (PHOTO) STORE AS SYS_LOB0000092017C00009$$
(
  ENABLE STORAGE IN ROW
  CHUNK 8192
  NOCACHE
  NOLOGGING
  TABLESPACE USER10
  STORAGE
  (
    INITIAL 106496
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
);
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211210698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



（3）创建PRODUCTS表
运行代码如下：

```sql
CREATE TABLE PRODUCTS
(
  PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
, PRODUCT_TYPE VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT PRODUCTS_PK PRIMARY KEY
  (
    PRODUCT_NAME
  )
  ENABLE
)
LOGGING
TABLESPACE "USER10"
PCTFREE 10
INITRANS 1
STORAGE
(
  INITIAL 65536
  NEXT 1048576
  MINEXTENTS 1
  MAXEXTENTS 2147483645
  BUFFER_POOL DEFAULT
);

ALTER TABLE PRODUCTS
ADD CONSTRAINT PRODUCTS_CHK1 CHECK
(PRODUCT_TYPE IN ('耗材', '手机', '电脑'))
ENABLE;
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211218537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



（4）创建ORDERS表
运行代码如下：

```sql
CREATE TABLE ORDERS
(
  ORDER_ID NUMBER(10, 0) NOT NULL
, CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL
, CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL
, ORDER_DATE DATE NOT NULL
, EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, DISCOUNT NUMBER(8, 2) DEFAULT 0
, TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0
)
TABLESPACE USER10
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NOPARALLEL
PARTITION BY RANGE (ORDER_DATE)
(
  PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USER10
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
  NOLOGGING
  TABLESPACE USER10
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
);
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211236646.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




（5）创建ORDER_DETAILS表
运行代码如下：

```sql
CREATE TABLE ORDER_DETAILS
(
  ID NUMBER(10, 0) NOT NULL
, ORDER_ID NUMBER(10, 0) NOT NULL
, PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
, PRODUCT_NUM NUMBER(8, 2) NOT NULL
, PRODUCT_PRICE NUMBER(8, 2) NOT NULL
, CONSTRAINT ORDER_DETAILS_FK1 FOREIGN KEY
  (
  ORDER_ID
  )
  REFERENCES ORDERS
  (
  ORDER_ID
  )
  ENABLE
)
TABLESPACE USER10
PCTFREE 10
INITRANS 1
STORAGE
(
  BUFFER_POOL DEFAULT
)
NOCOMPRESS
NOPARALLEL
PARTITION BY REFERENCE (ORDER_DETAILS_FK1)
(
  PARTITION PARTITION_BEFORE_2016
  NOLOGGING
  TABLESPACE USERS --必须指定表空间，否则会将分区存储在用户的默认表空间中
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY,
  PARTITION PARTITION_BEFORE_2017
  NOLOGGING
  TABLESPACE USER10
  PCTFREE 10
  INITRANS 1
  STORAGE
  (
    INITIAL 8388608
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOCOMPRESS NO INMEMORY
)
;
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211249163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



### 2.4插入5万条数据
运行代码如下：

```sql
declare
  dt date;
  m number(8,2);
  V_EMPLOYEE_ID NUMBER(6);
  v_order_id number(10);
  v_name varchar2(100);
  v_tel varchar2(100);
  v number(10,2);
begin
  for i in 1..10000
  loop
    if i mod 2 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60);
    else
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60);
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=SEQ_ORDER_ID.nextval; --应该将SEQ_ORDER_ID.nextval保存到变量中。
    v_name := 'aa'|| 'aa';
    v_name := 'zhang' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    --插入订单y一个订单包括3个产品
    v:=dbms_random.value(10000,4000);
    v_name:='computer'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,2,v);
    v:=dbms_random.value(1000,50);
    v_name:='paper'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,3,v);
    v:=dbms_random.value(9000,2000);
    v_name:='phone'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,1,v);
    --在触发器关闭的情况下，需要手工计算每个订单的应收金额：
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=v_order_id;
    if m is null then
     m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount WHERE ORDER_ID=v_order_id;
    IF I MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
  end loop;
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211442619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




### 2.5查询数据量
运行代码如下：

```sql
SELECT count(*) from ORDERS_DETAILS
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211459739.png)




### 2.6在数据库中建立一个程序包，在包中用PL/SQL语言设计一些存储过程和函数，实现比较复杂的业务逻辑，用模拟数据进行执行计划分析。
（1）建立程序包
运行代码如下：

```sql
create or replace PACKAGE MyPack IS
/*
    包MyPack中有：
  一个函数:Get_SaleAmount(V_DEPARTMENT_ID NUMBER)，
  一个过程:Get_Employees(V_EMPLOYEE_ID NUMBER)
  */
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END MyPack;
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211521708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



运行代码如下：

```sql
create or replace PACKAGE BODY MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
    N NUMBER(20,2); --注意，订单ORDERS.TRADE_RECEIVABLE的类型是NUMBER(8,2),汇总之后，数据要大得多。
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;
  PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                             V.EMPLOYEE_ID||' '||v.NAME);
      END LOOP;
    END;
END MyPack;
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211533500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)





（2）用模拟数据进行执行计划分析
运行代码如下：

```sql
select * from order_details od, orders ord,products pr where od.order_id=ord.ORDER_ID
and od.PRODUCT_NAME=pr.PRODUCT_NAME and ord.ORDER_DATE between to_date('2016-01-01','yyyy-mm-dd')
and to_date('2017-01-01','yyyy-mm-dd');
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211540362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)





### 2.7设计自动备份方案或则手工备份方案
（1）开始全备份
运行代码如下：

```sql
[oracle@oracle-pc ~]$ cat rman_level0.sh
[oracle@oracle-pc ~]$ ./rman_level0.sh
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211602279.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




（2）查看备份文件
*.log是日志文件
dblv0*.bak是数据库的备份文件
arclv0*.bak是归档日期的备份文件
c-1392946895-20191120-01是控制文件和参数的备份
[oracle@oracle-pc ~]$ cd rman_backup
[oracle@oracle-pc rman_backup]$ ls
arclv0_ORCL_20191120_dauhb2fm_1_1.bak
c-1392946895-20191120-01
dblv0_ORCL_20191120_d7uhb2ap_1_1.bak
dblv0_ORCL_20191120_d8uhb2c6_1_1.bak
dblv0_ORCL_20191120_d9uhb2ei_1_1.bak
lv0_20191120-083949_L0.log
（3）查看备份文件的内容
运行代码如下：

```sql
[oracle@oracle-pc ~]$ rman target /
RMAN> list backup;
```

运行结果截图：





![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112621161766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



由上面的"list backup;" 输出可以看出，备份集中的文件内容是：
/home/oracle/rman_backup/dblv0_ORCL_20191120_d7uhb2ap_1_1.bak 是插接数据库PDBORCL的备份集
/home/oracle/rman_backup/dblv0_ORCL_20191120_d8uhb2c6_1_1.bak 是ORCL的备份集
/home/oracle/rman_backup/dblv0_ORCL_20191120_d9uhb2ei_1_1.bak是PDB$SEED的备份集
/home/oracle/rman_backup/arclv0_ORCL_20191120_dauhb2fm_1_1.bak是归档文件的备份集
/home/oracle/rman_backup/c-1392946895-20191120-01是参数文件(SPFILE)和控制文件(Control File)的备份集
（4）备份后修改数据
运行代码如下：

```sql
[oracle@oracle-pc ~]$ sqlplus jh/123@pdborcl
SQL> create table t1 (id number,name varchar2(50));
SQL> insert into t1 values(1,'zhang');
SQL> commit;
SQL> select * from t1;
SQL> exit
```

运行结果截图：



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211630596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)





（5）删除数据库文件，模拟数据库文件损坏

```sql
[oracle@oracle-pc~]$ rm/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_user10_1.pdf
```

（6）删除数据库文件后修改数据
运行代码如下：

```sql
[oracle@oracle-pc ~]$ sqlplus jh/123@pdborcl
SQL> insert into t1 values(2,'wang');
SQL> commit;
SQL> select * from t1;
SQL> declare
  2  n number;
  3  begin
  4    for n in 1..10000 loop
  5      insert into t1 values(n,'name'||n);
  6    end loop;
  7  end;
  8  /
declare
*
SQL> select * from t1;
SQL> exit
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211644603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




（7）重启损坏的数据库到mount状态
运行代码如下：

```sql
[oracle@oracle-pc ~]$ sqlplus / as sysdba
SQL> shutdown immediate
SQL> shutdown abort
SQL> startup mount
SQL> exit
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211653804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)




（8）开始恢复数据库
运行代码如下：

```sql
[oracle@oracle-pc ~]$ rman target /
RMAN> restore database ;
RMAN> recover database;
RMAN> alter database open;
RMAN> exit
```

运行结果截图：






![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211707342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211713954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)





（9）查询数据是否恢复
运行代码如下：

```sql
[oracle@oracle-pc ~]$ sqlplus jh/123@pdborcl
SQL> select * from t1;
```

运行结果截图：




![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126211724655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)



由以上查询结果可见，数据100%恢复了。
### 2.8设计容灾方案。使用两台主机，通过DataGuard实现数据库整体的异地备份。
第一步：备库

```sql
mkdir -p /home/oracle/app/oracle/diag/orcl
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdborcl
mkdir -p /home/oracle/arch
mkdir -p /home/oracle/rman
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdbseed/
mkdir -p /home/oracle/app/oracle/oradata/stdorcl/pdb/
//linux 命令，一行一行在终端运行
//删除原有数据库:
$sqlplus / as sysdba         //连数据库
shutdown immediate;          //sql>
startup mount exclusive restrict;      //sql>
drop database;              //sql>

exit;                    //sql>
启动到nomount
$sqlplus / as sysdba         //连数据库
startup nomount             //sql>
sql>
//换到primary
```

第二步：主库:

```sql
$sqlplus /  sysdba              //sql>
select group#,thread#,members,status from v$log;          //sql>
alter database add standby logfile  group 5 '/home/oracle/app/oracle/oradata/orcl/stdredo1.log' size 50m; //sql>
alter database add standby logfile  group 6 '/home/oracle/app/oracle/oradata/orcl/stdredo2.log' size 50m; //sql>
alter database add standby logfile  group 7 '/home/oracle/app/oracle/oradata/orcl/stdredo3.log' size 50m; //sql> 
alter database add standby logfile  group 8 '/home/oracle/app/oracle/oradata/orcl/stdredo4.log' size 50m; //sql>
主库环境开启强制归档
ALTER DATABASE FORCE LOGGING;  //sql>
alter system set LOG_ARCHIVE_CONFIG='DG_CONFIG=(orcl,stdorcl)' scope=both sid='*';         //sql>
alter system set log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=orcl' scope=spfile;
alter system set LOG_ARCHIVE_DEST_2='SERVICE=stdorcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=stdorcl' scope=both sid='*';
alter system set fal_client='orcl' scope=both sid='*';    
alter system set FAL_SERVER='stdorcl' scope=both sid='*';  
alter system set standby_file_management=AUTO scope=both sid='*';
alter system set DB_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';  
alter system set LOG_FILE_NAME_CONVERT='/home/oracle/app/oracle/oradata/stdorcl/','/home/oracle/app/oracle/oradata/orcl/' scope=spfile sid='*';
alter system set log_archive_format='%t_%s_%r.arc' scope=spfile sid='*';
alter system set remote_login_passwordfile='EXCLUSIVE' scope=spfile;
alter system set PARALLEL_EXECUTION_MESSAGE_SIZE=8192 scope=spfile;
exit;
```

编辑主库以及备库的/home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora文件

```sql
$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/tnsnames.ora  //terminal
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.206.131)(PORT = 1521))  //
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
stdorcl =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.206.132)(PORT = 1521))  //
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SID = orcl)
    )
  )
```

  //ifconfig 查询ip后 测试两能否平ping通  只改ip地址   最后一个块被替换为以上两个块  记得保存（主+备）
在主库上生成备库的参数文件:
$sqlplus /  as sysdba   //如果没有出现sql>
SQL>create pfile from spfile;
生成/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora
exit;
将主库的参数文件，密码文件拷贝到备库:

```sql
scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora 192.168.206.132:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/
scp /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/orapworcl 192.168.206.132:/home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/
```

将主库复制到备库

```sql
$rman target sys/123@orcl auxiliary sys/123@stdorcl      //terminal运行
rman>
执行duplicate:
run{ 
allocate channel c1 type disk;
allocate channel c2 type disk;
allocate channel c3 type disk;
allocate AUXILIARY channel c4 type disk;
allocate AUXILIARY channel c5 type disk;
allocate AUXILIARY channel c6 type disk;
DUPLICATE TARGET DATABASE
  FOR STANDBY
  FROM ACTIVE DATABASE
  DORECOVER
  NOFILENAMECHECK;
release channel c1;
release channel c2;
release channel c3;
release channel c4;
release channel c5;
release channel c6;
}
//等待直至出现rman>
```

第三步：备库
在备库上更改参数文件
$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/dbs/initorcl.ora  //运行
文件内容是：
orcl.__data_transfer_cache_size=0
orcl.__db_cache_size=671088640
orcl.__java_pool_size=16777216
orcl.__large_pool_size=33554432
orcl.__oracle_base='/home/oracle/app/oracle'#ORACLE_BASE set from environment
orcl.__pga_aggregate_target=536870912
orcl.__sga_target=1258291200
orcl.__shared_io_pool_size=50331648
orcl.__shared_pool_size=301989888
orcl.__streams_pool_size=0
*._allow_resetlogs_corruption=TRUE
*._catalog_foreign_restore=FALSE
*.audit_file_dest='/home/oracle/app/oracle/admin/orcl/adump'
*.audit_trail='db'
*.compatible='12.1.0.2.0'
*.control_files='/home/oracle/app/oracle/oradata/orcl/control01.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control02.ctl','/home/oracle/app/oracle/fast_recovery_area/orcl/control03.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.db_name='orcl'
*.db_unique_name='stdorcl'
*.db_recovery_file_dest='/home/oracle/app/oracle/fast_recovery_area'
*.db_recovery_file_dest_size=4823449600
*.diagnostic_dest='/home/oracle/app/oracle'
*.dispatchers='(PROTOCOL=TCP)(dispatchers=1)(pool=on)(ticks=1)(connections=500)(sessions=1000)'
*.enable_pluggable_database=true
*.fal_client='stdorcl'
*.fal_server='orcl'
*.inmemory_max_populate_servers=2
*.inmemory_size=157286400
*.local_listener=''
*.log_archive_config='DG_CONFIG=(stdorcl,orcl)'
*.log_archive_dest_1='LOCATION=/home/oracle/arch VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=stdorcl'
*.log_archive_dest_2='SERVICE=orcl LGWR ASYNC  VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=orcl'
*.log_archive_format='%t_%s_%r.arc'
*.log_file_name_convert='/home/oracle/app/oracle/oradata/orcl/','/home/oracle/app/oracle/oradata/stdorcl/'
*.max_dispatchers=5
*.max_shared_servers=20
*.open_cursors=400
*.parallel_execution_message_size=8192
*.pga_aggregate_target=511m
*.processes=300
*.recovery_parallelism=0
*.remote_login_passwordfile='EXCLUSIVE'
*.service_names='ORCL'
*.sga_max_size=1572864000
*.sga_target=1258291200
*.shared_server_sessions=200
*.standby_file_management='AUTO'
*.undo_tablespace='UNDOTBS1'
//以上全部替换弹出的文件内容    记得保存
在备库增加静态监听
$gedit /home/oracle/app/oracle/product/12.1.0/dbhome_1/network/admin/listener.ora //运行
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (ORACLE_HOME = /home/oracle/app/oracle/product/12.1.0/db_1)
      (SID_NAME = orcl)
    )
  )
 //添加到此文件最后 
重新启动,备库开启实时应用模式:：

```sql
$sqlplus / as sysdba //再次连数据库
shutdown immediate   //sql>
startup              //sql>
alter database recover managed standby database disconnect;        //sql>
```

测试：
在主库连数据库
$sqlplus / as sysdba //连数据库
create table hugu(sex varchar);
进入备库：
$sqlplus / as jhdba //连数据库
select * from jh;






















