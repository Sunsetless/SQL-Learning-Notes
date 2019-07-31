SQL语言学习——DML部分
----------

——作者：Su

- 数据操作语言
  - 插入：`insert`
  - 修改：`update`
  - 删除：`delete`

## 1. 插入语句

### 1）方式一

```mysql
insert into 表名(列名1,列名2……)
values(值1,值2……),(值1,值2……);
```

- 插入的值的类型要与列的类型一致或兼容

  ```mysql
  INSERT INTO beauty(id,name,sex,borndate,phone,photo,boyfriend_id)
  VALUES(13,'唐艺昕','女','1990-4-23','189888888',NULL,2);
  ```

- 不可以为null的列必须插入值，可以为null的可直接输入`NULL`或在`insert`语句后面不列出该字段

  ```mysql
  INSERT INTO beauty(id,name,sex,borndate,phone,boyfriend_id)
  VALUES(13,'唐艺昕','女','1990-4-23','189888888',2);
  ```


- `insert`语句后面列的顺序可以不按照表中的顺序

  ```mysql
  INSERT INTO beauty(name,sex,id,phone)
  VALUES('关晓彤','女',17,'110');
  ```

- 可以省略列名，默认所有列，列的顺序与表中列的顺序一致

  ```mysql
  INSERT INTO beauty
  VALUES(18,'张飞','男',NULL,'119',NULL,NULL);
  ```

### 2）方式二

```mysql
insert into 表名
set 列名=值,列名=值……
```

```mysql
INSERT INTO beauty
SET id=19,name='刘涛',phone="999";
```

### 3）方式一与方式二对比

1. 方式一支持插入多行，方式二不支持

   ```mysql
   INSERT INTO beauty(id,name,sex,borndate,phone,boyfriend_id)
   VALUES(23,'唐艺昕1','女','1990-4-23','189888888',2),
   (24,'唐艺昕2','女','1990-4-23','189888888',2),
   (25,'唐艺昕3','女','1990-4-23','189888888',2)
   ```

2. 方式一支持子查询，方式二不支持

   ```mysql
   INSERT INTO beauty(id,name,phone)
   SELECT 26,'宋茜','119123039'
   ```

   将`select`后的信息插入表中

   ```mysql
   INSERT INTO beauty(id,name,phone)
   SELECT id,boyname,'1234243'
   FROM boys WHERE id<3;
   ```

   `select`可以使用`union`连接

   ```mysql
   INSERT INTO my_employees
   SELECT 1,'patel','Ralph','Rpatel',895 UNION
   SELECT 2,'Dancs','Betty','Bdancs',860 UNION
   SELECT 3,'Biri','Ben','Bbiri',1100;
   ```

   

## 2. 修改语句

```mysql
UPDATE 表名
SET 列=新值,列=新值……
WHERE 筛选条件;
```

### 1）修改单表条目

- 案例1

  修改beauty表中姓唐的女神的电话为13899888899

  ```mysql
  UPDATE beauty
  SET phone=13899888899
  WHERE `name` LIKE "唐%";
  ```

- 案例2

  修改boys表中id号为2的名称为张飞，魅力值10

  ```mysql
  UPDATE boys
  SET boyName="张飞" ,userCP=10
  WHERE id=2;
  ```

### 2）修改多表条目

- SQL92

  ```mysql
  UPDATE 表1 别名,表2 别名
  SET 列=值,列=值……
  WHERE 连接条件
  AND	筛选条件
  ```

- SQL99

  ```mysql
  UPDATE 表1 别名
  INNER|LEFT|RIGHT JOIN 表2 别名
  ON 连接条件
  SET 列=值,列=值……
  WHERE 筛选条件
  ```

- 案例1

  修改张无忌的女朋友的手机号为114

  ```mysql
  UPDATE beauty b
  INNER JOIN boys bo
  ON b.boyfriend_id=bo.id
  SET b.phone='114'
  WHERE bo.boyname='张无忌'
  ```

- 案例2

  修改没有男朋友的男朋友编号都为2

  ```mysql
  UPDATE beauty b
  LEFT JOIN boys bo
  ON b.boyfriend_id=bo.id
  SET b.boyfriend_id=2
  WHERE bo.id IS NULL -- bo.id是主键，主键为null，则表示没有对应条目
  ```

## 3. 删除语句

### 1）方式一：delete

#### 单表的条目删除

`delete from 表名 【where 筛选条件】【limit 条目数】`

清空表，`delete from 表名;`

- 案例1

  删除手机号以9结尾的女神信息

  ```mysql
  DELETE FROM beauty WHERE phone LIKE "%9";
  ```

#### 多表的条目删除

SQL92：

```mysql
DELETE 表1的别名，表2的别名
FROM 表1 别名,表2 别名
WHERE 连接条件
AND 筛选条件;
```

SQL99：

```mysql
DELETE 表1的别名，表2的别名
FROM 表1 别名
INNER|LEFT|RIGHT JOIN 表2 别名
WHERE 筛选条件;
```

- 案例1

  删除张无忌的女朋友的信息

  ```mysql
  DELETE b
  FROM beauty b
  INNER JOIN boys bo
  ON b.boyfriend_id=bo.id
  WHERE bo.boyname="张无忌";
  ```

- 案例2

  删除黄晓明的信息以及他女朋友的信息

  ```mysql
  DELETE b,bo
  FROM beauty b
  INNER JOIN boys bo
  ON b.boyfriend_id=bo.id
  WHERE bo.boyName="黄晓明";
  ```

### 2）方式二：truncate

`truncate table 表名`清空表

`truncate`后不可接`where`

### 3）方式一与方式二对比

1. `delete`可以加`where`条件，`truncate`不可以
2. `truncate`删除，效率高一些
3. 如果要删除的表中有自增长列，如果用delete删除，再插入数据，自增长列的值从断点开始，而`truncate`删除后，再插入数据，自增长列的值从1开始
4. `truncate`删除没有返回值，`delete`删除有返回值（返回删除了几行）
5. `truncate`删除不能回滚，`delete`删除可以回滚

