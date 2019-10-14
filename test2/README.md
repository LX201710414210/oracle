## 实验二
###  第1步：以system登录到pdborcl，创建角色LX和用户LX123，并授权和分配空间：
代码结果显示：

SQL> CREATE ROLE LX1;
Role created.
SQL> GRANT connect,resource,CREATE VIEW TO LX1;
Grant succeeded.
SQL> CREATE USER LX2 IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
User created.
SQL> ALTER USER LX2r QUOTA 50M ON users;
User altered.
SQL> GRANT LX1 TO LX2;
Grant succeeded.
SQL> exit
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013175833239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)
### 第2步：新用户new_user连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
代码结果显示：

SQL> show user;
USER is "LX2"
SQL> CREATE TABLE mytable (id number,name varchar(50));
Table created.
SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');
1 row created.
SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');
1 row created.
SQL> CREATE VIEW myview AS SELECT name FROM mytable;
View created.
SQL> SELECT * FROM myview;
NAME
--------------------------------------------------
zhang
wang
SQL> GRANT SELECT ON myview TO hr;
Grant succeeded.
SQL>exit
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013180436655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)
### 第3步：用户hr连接到pdborcl，查询new_user授予它的视图myview。
代码结果显示：

SQL> SELECT * FROM LX2.myview;
NAME
--------------------------------------------------
zhang
wang
SQL> exit
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013180727309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)
### 第四步：查看数据库的使用情况。
代码结果显示：

SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';
SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191013181113793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDQyMjYw,size_16,color_FFFFFF,t_70)
