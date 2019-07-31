# SQL语言学习——TCL部分

**TCL：Transaction Control Language**

事务：一个或一组SQL语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行

案例：转账

使用`SHOW ENGINES`查看MySQL中支持的存储引擎

在MySQL中用的最多的存储引擎有：innodb，myisam，memory等，其中innodb支持事务，而myisam、memory等不支持事务

## 事务的ACID属性

1. 原子性（Atomicity）

   原子性是指事务时一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生

2. 一致性（Consistency）

   事务必须使数据库从一个一致性状态变换到另外一个一致性状态

3. 隔离性（Isolation）

   事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务时隔离的，并发执行的各个事务之间不能互相干扰

4. 持久性（Durability）

   持久性是指一个事务一旦被提交，它对数据库中的数据的改变就是永久性的，接卸来的其他操作和数据库故障不应该对其有任何影响

## 事务的创建

- 隐式事务：事务没有明显的开启和结束的标记

  如`insert`、`update`、`delete`语句

- 显式事务：事务具有明显的开启和结束的标记

  前提：必须先设置自动提交功能为禁用，`set autocommit=0;`

  ​	*每次开启显示事务都需要运行`set autocommit=0;`*

  步骤1：开启事务

  ​	`set autocommit=0;`

  ​	`start transaction;`可选

  步骤2：编写事务中的SQL语句（select、insert、update、delete）

  ​	语句1;

  ​	语句2;

  步骤3：结束事务

  ​	`commit;`提交事务

  ​	`rollback;`回滚事务

  ​	`savepoint 回滚点名;`设置保存点,与`rollback to 回滚点名;`搭配使用

- 案例：转账

  ```mysql
  -- 创建表格，id、用户名、账户
  CREATE TABLE account(
  	id INT PRIMARY KEY auto_increment,
  	username VARCHAR(20),
  	balance DOUBLE
  );
  -- 插入数据
  INSERT INTO account(username,balance)
  VALUES('john',1000),('mike',1000);
  
  -- 开始事务
  SET autocommit=0;
  START TRANSACTION;
  -- 编写一组事务的语句
  UPDATE account SET balance=500 where username='john';
  UPDATE account SET balance=1500 WHERE username="mike";
  -- 结束事务
  COMMIT;
  -- 或者是 ROLLBACK;
  ```

## 数据库的隔离级别

- 数据库事务的隔离性：数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题
- 对于同时运行的多个事务，当这些事务访问数据库中的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：
  - 脏读：对于两个事务T1、T2，T1读取了已经被T2更新但还**没有被提交**的字段。之后，若T2回滚，T1读取的内容就是临时且无效的
  - 不可重复读：对于两个事务T1、T2，T1读取了一个字段，然后T2**更新**了该字段。之后，T1再次读取同一个字段，值就不同了
  - 幻读：对于两个事务T1、T2，T1从表中读取了一个字段，然后T2在该表中**插入**了一些新的行。之后，如果T1再次读取同一个表，就会多出几行
- 一个事务与其他事务隔离的程度称为隔离级别，数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，**隔离级别越高，数据一致性就越好，但并发性越弱**
- 数据库提供的4种事务隔离级别：
  - **READ UNCOMMITTED（读未提交数据）**：允许事务读取未被其他事务提交的变更。脏读、不可重复读和幻读的问题都会出现
  - **READ COMMITTED（读已提交数据）**：只允许事务读取已经被其它事务提交的变更，可以避免脏读，但不可重复读和幻读问题仍然可能出现
  - **REPEATABLE READ（可重复读）**：确保事务可以多此从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新，可以避免脏读、不可重复读，但幻读的问题仍然存在
  - **SERIALIZABLE（串行化）**：确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作，所有并发问题都可以避免，**但性能十分低下**
- Oracle支持2种事务隔离级别：**READ COMMITTED、SERIALIZABLE**。默认为READ COMMITTED。
- MySQL支持4中事务隔离级别，默认为：**REPEATABLE READ**



- 查看当前隔离级别，`select @@tx_isolation;`

  MySQL 8.0.3之后，变量被transaction_isolation替换，`select @@transaction_isolation;`或`show variables like 'transaction_isolation';`

- 设置当前连接的隔离级别，` set session transaction isolation level read uncommitted;`
- 设置全局隔离级别，` set global transaction isolation level read uncommitted;`

# 视图

MySQL从5.0.1开始提供视图功能，一种虚拟存在的表，行和列的数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的，只保存了SQL逻辑，不保存查询结果

- 应用场景
  - 多个地方用到同样的查询结果
  - 该查询结果使用SQL语句较复杂

## 一、视图的创建

- 案例：查询姓张的学生名和专业名

  ```mysql
  SELECT stuname,majorname
  FROM stuinfo s
  INNER JOIN major m ON s.majorid=m.id
  WHERE s.stuname LIKE '张%';
  
  -- 以上代码可以用以下形式表达 
  CREATE VIEW v1
  AS
  SELECT stuname,majorname
  FROM stuinfo s
  INNER JOIN major m ON s.majorid=m.id
  
  SELECT * FROM v1
  WHERE s.stuname LIKE '张%';
  ```

**优点：**

- 重用SQL语句
- 简化复杂的SQL操作，不必知道它的查询细节
- 保护数据，提高安全性，因为无法查询到原始表中的各字段

## 二、视图的修改

- 方式一：`create or replace view 视图名 as ...`
- 方式二：`alter view 视图名 as ...`

## 三、视图的删除

- `drop view 视图名,视图名,...;`

## 四、查看视图

- `desc 视图名;`

## 五、视图的更新

- insert / update / delete，与对表的编辑一致
- 更新效果会应用在原始表，因此可能造成数据的不安全
- **具备以下特点的视图不允许更新**
  - 包含以下关键字的SQL语句：分组函数、distinct、group by、having、union、union all
  - 常量视图
  - select中包含子查询
  - join语句
  - from一个不能更新的视图
  - where子句中的子查询引用了from子句中的表

# 变量

- 系统变量
  - 全局变量
  - 会话变量

- 自定义变量
  - 用户变量
  - 局部变量

## 一、系统变量

变量由系统提供，不是用户定义，属于服务器层面的语法

注意：如果是全局级别，则需要加global，如果是会话级别，则需要加session，如果不写，则默认session

- 查看所有的系统变量

  `show global|[session] variables;`

- 查看满足条件的部分系统变量，如：

  `show global|[session] variables like '%char%';`

- 查看指定的某个系统变量的值

  `select @@global|[session].系统变量名`

- 为某个系统变量赋值

  `set global|[session] 系统变量名 = 值`

  `set @@global|session.系统变量名 = 值` ，必须有`session`

### 1. 全局变量

作用域：服务器每次启动将为所有的全局变量赋初始值，针对于所有的**会话（连接）**有效，但不能跨重启。*若想跨重启，则需修改配置文件*

```mysql
SHOW GLOBAL VARIABLES;

SHOW GLOBAL VARIABLES LIKE '%char%'

SELECT @@global.autocommit;
SELECT @@transaction_isolation;

SET @@global.autocommit=0;
```

### 2. 会话变量

作用域：仅仅针对当前**会话（连接）**有效

## 二、自定义变量

变量由用户自定义，不是由系统产生

### 1. 用户变量

作用域：针对于当前会话（连接）有效

应用在任何地方，即begin end之中或之外

赋值操作符：`=`或`:=`

1. 声明并初始化

   `set @用户变量名=值;`

   或 `set @用户变量名:=值;`

   或 `select @用户变量名:=值;`

2. 赋值（更新用户变量的值）

   方式一：通过`SET`或者`SELECT`

   ​	`set @用户变量名=值;`

   ​	或 `set @用户变量名:=值;`

   ​	或 `select @用户变量名:=值;`

   方式二：

   ​	`select 字段 into 变量名 from 表;`

3. 使用（查看用户变量的值）

   `select @用户变量名;`

```mysql
-- 声明并初始化
SET @name='john';
SET @name=100;
SET @count=1;
-- 赋值
SELECT COUNT(*) INTO @count FROM employees;
-- 查看
SELECT @count;

SET @m=1;
SET @n=2;
SET @sum = @m+@n;
SELECT @sum; -- 结果为3
```

### 2. 局部变量

作用域：仅仅在定义的begin end中有效

应用begin end 中的**第一句话**

1. 声明

   `DECLARE 变量名 类型;`
   `DECLARE 变量名 类型 DEFAULT 值;`

2. 赋值

   方式一：通过`SET`或者`SELECT`

   ​	`set 用户变量名=值;`

   ​	或 `set 用户变量名:=值;`

   ​	或 `select @用户变量名:=值;`，若使用select，则后面必须有【@】

   方式二：通过``select into`

   ​	`select 字段 into 局部变量名 from 表;`

3. 使用

   `select 局部变量名;`

# 存储过程和函数

类似于java中的方法

## 一、存储过程

含义：一组预先编译好的SQL语句的集合，批处理语句

- 优点：
   	1. 提高代码的重用性
      	2. 简化操作
              	3. 减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

### 1. 创建语法

```mysql
CREATE PROCEDURE 存储过程名(参数列表)

BEGIN
	存储过程体(一组合法的SQL语句)
END
```

1. 参数列表包含三部分：参数模式、参数名、参数类型，如：

   `IN stuname VARCHAR(20)`

   参数模式：

   - IN：该参数可以作为输入，即该参数需要调用方法传入值

     `CALL sp1（‘值’）`

   - OUT：该参数可以作为输出，即该参数可以作为返回值

     `SET @name; CALL sp1(@name); SELECT @name;`

   - INOUT：该参数既可以作为输入又可以作为输出，即该参数既需要传入值，又可以返回值

     `SET @name=值; CALL sp1(@name); SELECT @name;`

2. 如果存储过程体只有一句话，BEGIN END可以省略

   存储过程体中的每条SQL语句的结尾必须有【;】

   存储过程的结尾可以使用`DELIMITER 结束标记`重新设置，如：

   `DELIMITER $`

### 2. 调用语法

`CALL 存储过程名（实参列表）`

- 案例1：空参列表

  插入到major表中5条记录

  ```mysql
  DELIMITER $
  CREATE PROCEDURE myp1()
  BEGIN
  	INSERT INTO major
  	VALUES(5,'C'),(6,'go'),(7,'python'),(8,'win'),(9,'C++');
  END $
  
  CALL myp1()$
  ```

- 案例2：IN模式参数的存储过程

  创建存储过程实现 根据女神名，查询对应的男神信息

  ```mysql
  CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
  BEGIN
  	SELECT bo.*
  	FROM boys.bo
  	RIGHT JOIN beauty b ON bo.id=b.boyfriend_id
  	WHERE b.name=beautyName; -- beautyName为形参名
  END $
  
  CALL myp2('赵敏')$
  ```

- 案例3：IN模式参数的存储过程

  创建存储过程实现 判断用户是否登录成功

  ```mysql
  CREATE PROCEDURE myp4(IN username VARCHAR(20),IN password VARCHAR(20))
  BEGIN
  	DECLARE result INT DEFAULT 0; -- 声明变量result
  	SELECT COUNT(*) INTO result; -- 给result变量赋值
  	FROM admin
  	WHERE admin.username=username
  	AND admin.password=password;
  	SELECT IF(result>0,'登录成功','登录失败');
  END $
  
  CALL myp4('张飞','6666') $
  ```

- 案例4：OUT模式参数的存储过程

  创建存储过程实现 根据女神名，返回对应的男神信息

  ```mysql
  CREATE PROCEDURE myp5(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20))
  BEGIN
  	SELECT bo.boyName INTO boyName
  	FROM boys bo
  	INNER JOIN beauty b ON bo.id=b.boyfriend_id
  	WHERE b.name=beautyName;
  END $
  
  SET @bName $ -- 此行可省略
  CALL myp5('王语嫣',@bName)$
  SELECT @bName $
  ```

- 案例5：OUT模式参数的存储过程

  根据女神名，返回对应的男神名和魅力值

  ```mysql
  CREATE PROCEDURE myp6(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT userCP INT)
  BEGIN
      SELECT bo.boyName,bo.userCP INTO boyName,userCP
      FROM boys bo
      INNER JOIN beauty b ON b.boyfriend_id=bo.id
      WHERE b.name=beautyName;
  END $
  
  CALL myp6('王语嫣',@bName,@userCP) $
  SELECT @bName,@userCP $
  ```

- 案例6：INOUT模式参数的存储过程

  传入a和b两个值，最终a和b都翻倍并返回

  ```mysql
  CREATE PROCEDURE myp7(INOUT a INT,INOUT b INT)
  BEGIN
  	SET a=a*2
  	SET b=b*2
  END $
  
  SET @m=10$
  SET @n=20$
  CALL myp7(@m,@n)$
  SELECT @m,@n $
  ```

### 3. 删除存储过程

`DROP PROCEDURE 存储过程名`

```mysql
DROP PROCEDURE p1；
DROP PROCEDURE p2，p3；-- 此行**错误**，不可以批量删除
```

### 4. 查看存储过程的信息

`SHOW CREATE PROCEDURE test2;`

### 5. 练习题

1. 创建存储过程实现传入用户名和密码，插入到admin表中

   ```mysql
   CREATE PROCEDURE test1(IN username VARCHAR(20),IN pwd VARCHAR(20))
   BEGIN
   	INSERT INTO admin(admin.username,password)
   	VALUES(username,pwd);
   END $
   ```

2. 创建存储过程实现传入女神编号，返回女神名称和女神电话

   ```mysql
   CREATE PROCEDURE test2(IN beautyID INT,OUT beautyName VARCHAR(20),OUT phone VARCHAR(20))
   BEGIN
       SELECT b.name,b.phone INTO beautyName,phone
       FROM beauty b
       WHERE b.id=beautyID;
   END $
   
   CALL test2(1,@n,@p)
   SELECT @n,@p
   ```

   另一种方式如下：

   ```mysql
   CREATE PROCEDURE test_2(IN beautyID INT,OUT beautyName VARCHAR(20),OUT phone VARCHAR(20))
   BEGIN
       SELECT b.name,b.phone INTO beautyName,phone
       FROM beauty b
       WHERE b.id=beautyID;
       SELECT beautyName,phone; -- 在存储过程定义中加入此行，则调用后直接有返回结果
   END $
   
   CALL test2(1,@n,@p)
   ```

3. 创建存储过程或函数实现传入两个女生生日，返回大小

   ```mysql
   CREATE PROCEDURE test3(IN borndate_1 DATETIME,IN borndate_2,DATETIME,OUT result INT)
   BEGIN
   	SELECT DATEDIFF(borndate_1,borndate_2) INTO result;
   END $
   ```

4. 创建存储过程或函数实现传入一个日期，格式化为xx年xx月xx日并返回

   ```mysql
   CREATE PROCEDURE test4(IN date DATETIME,OUT strDATE VARCHAR(50))
   BEGIN
   	SELECT DATE_FORMAT(date,'%y年%m月%d日') INTO strDATE;
   END $
   
   CALL test4(NOW(),@str) $
   SELECT @str $
   ```

5. 创建存储过程或函数实现传入女神姓名，返回：女神 AND 男神，格式化的字符串

   ```mysql
   CREATE PROCEDURE test5(IN beautyName VARCHAR(20),OUT str VARCHAR(50))
   BEGIN
   	SELECT CONCAT(beautyName,' AND ',IFNULL(boyName,'null')) INTO srt
   	FROM boys
   	RIGHT JOIN beauty b ON b.boyfriend_id = boys.id
   	WHERE b.name=beautyName
   END $
   
   CALL test5('刘岩'，@str) $
   SELECT @str $
   ```

6. 创建存储过程或函数，根据传入的条目数和起始索引，查询beauty表的记录

   ```mysql
   CREATE PROCEDURE test6(IN startIndex INT,IN size INT)
   BEGIN
   	SELECT * FROM beauty LIMIT startIndex,size;
   END $
   
   CALL test6(3,5) $ -- 起始索引3，条目数5
   ```

## 二、函数

- 含义：一组预先编译好的SQL语句的集合，理成批处理语句

1. 提高代码的重用性
2. 简化操作
3. 减少了编译次数，并且减少了和数据库服务器的连接次数，提高了效率

- 与存储过程区别：
  - 存储过程：可以有0个返回，也可以有多个返回。适合做批量插入、批量更新
  - 函数：有且仅有1个返回，适合做处理数据后返回一个结果

### 1. 创建语法

```mysql
CREATE FUNCTION 函数名（参数列表） RETURNS 返回类型
BEGIN
	函数体
END
```

1. *参数列表包括【参数名】和【参数类型】*

2. 函数体：肯定有`RETURN`语句，否则会报错；若`RETURN`语句没有在函数体的最后，不报错，但不建议
3. 函数体中若仅有一句话，则可以省略`BEGIN`和`END`
4. 使用`delimiter`语句设置结束标记

### 2. 调用语法

```mysql
SELECT 函数名（参数列表）
```

### 3. 案例

#### （一）无参有返回

- 案例1：返回公司的员工个数

  ```mysql
  CREATE FUNCTION myf1() RETURNS INT
  BEGIN
  	DECLARE c INT DEFAULT 0;
  	SELECT COUNT(*) INTO c
  	FROM employees;
  	RETURN c;
  END $
  
  SELECT myf1() $
  ```

  *在命令行中执行创建函数的语句的时候可能报错：*

  `ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)`

  *可以执行`set global log_bin_trust_function_creators=TRUE;`解决*

#### （二）有参有返回

- 案例2：根据员工名，返回他的工资

  ```mysql
  CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
  BEGIN
  	SET @sal=0;  -- 此处设置了用户变量
  	SELECT salary INTO @sal
  	FROM employees
  	WHERE last_name = empName;
  	RETURN @sal;
  END $
  
  SELECT myf2('Chen') $
  SELECT @sal $ -- 由于@sal为用户变量，此语句仍然可以查询到薪资
  ```

- 案例3：根据部门名，返回该部门的平均工资

  ```mysql
  CREATE FUNCTION myf_3(depName VARCHAR(20)) RETURNS DOUBLE
  BEGIN
  	DECLARE avgSal DOUBLE;
  	SELECT AVG(salary) INTO avgSal
  	FROM employees e
  	JOIN departments d ON e.department_id = d.department_id
  	WHERE d.department_name=depName;
  	RETURN avgSal;
  END $
  
  SELECT myf3('IT') $
  ```

  *以上为标准答案，同时可以使用子查询，示例如下：*

  ```mysql
  CREATE FUNCTION myf3(depName VARCHAR(20)) RETURNS DOUBLE
  BEGIN
  	DECLARE avgSal DOUBLE;
  	SELECT AVG(salary) INTO avgSal
  	FROM employees
  	WHERE department_id IN (
  		SELECT department_id FROM departments
  		WHERE department_name = depName
  		);
  	RETURN avgSal;
  END $
  ```

#### （三）两个参数

- 案例4：创建函数，实现传入连个float，返回二者之和

  ```mysql
  CREATE FUNCTION myf4(num1 FLOAT,num2 FLOAT) RETURNS FLOAT
  BEGIN
  	DECLARE sum0 FLOAT;
  	SET sum0 = num1+num2;
  	RETURN sum0;
  END $
  
  SELECT myf4(1,2) $
  ```

### 4. 查看函数

`SHOW CREATE FUNCTION myp3;`

### 5. 删除函数

`DROP FUNCTION myp3;`

# 流程控制函数

- 顺序结构：程序从上往下依次执行

- 分支结构：程序从两条或多条路径中选择一条去执行

- 循环结构：程序在满足一定条件的基础上，重复执行一段代码

## 一、分支结构

### 1. if函数

- 功能：实现简单的双分支
- 语法 `IF （表达式1，表达式2，表达式3）`
- 执行顺序：如果表达式1成立，则IF函数返回表达式2的值，否则返回表达式3的值。
- 可以作为表达式放在任何位置

### 2. case结构

- 作为独立语句：只能放在BEGIN END中

  ```mysql
  CASE 变量 | 表达式 | 字段
      WHEN 要判断的值1 THEN 语句1;
      WHEN 要判断的值2 THEN 语句2;
      。。。
      ELSE 语句n;
  END CASE;
  ```

  ```mysql
  CASE
      WHEN 要判断的条件1 THEN 语句1;
      WHEN 要判断的条件2 THEN 语句2;
      。。。
      ELSE 语句n;
  END CASE;
  ```

- 作为表达式：嵌套在其他语句中使用，可以放在任何地方，BEGIN END中或外面

  ```mysql
  CASE 变量 | 表达式 | 字段
      WHEN 要判断的值 THEN 返回的值1
      WHEN 要判断的值 THEN 返回的值2
      。。。
      ELSE 返回的值n
  END;
  ```

  ```mysql
  CASE
      WHEN 要判断的条件1 THEN 返回的值1
      WHEN 要判断的条件1 THEN 返回的值2
      。。。
      ELSE 返回的值n
  END;
  ```

- 特点：

  - 可以作为表达式，嵌套在其他语句中使用，可以放在任何地方，BEGIN END中或外面

  - 可以作为独立的语句使用，只能放在BEGIN END中
  - 如果WHEN中的值满足或条件成立，则执行对应的THEN后面的语句，并且结束CASE。如果都不满足，则执行ELSE中的语句或值
  - ELSE可以省略，如果省略后所有WHEN条件都不满足，则返回NULL

- 案例：创建存储过程，根据传入的成绩，来显示等级，A：90-100，B：80-90，C：60-80，D：else

  ```mysql
  CREATE PROCEDURE test_case(IN score INT)
  BEGIN
  	CASE
          WHEN score>=90 AND score<=100 THEN SELECT 'A';
          WHEN score>=80 THEN SELECT 'B';
          WHEN score>=60 THEN SELECT 'C';
          ELSE SELECT 'D';
  	END CASE;
  END $
  
  CALL test_case(95) $
  ```

### 3. if结构

功能：实现多重分支

```MYSQL
IF 条件1 THEN 语句1;
ELSEIF 条件2 THEN	语句2;
。。。
ELSE 语句n;
END IF;
```

- 案例：根据传入的成绩，来显示等级，A：90-100，B：80-90，C：60-80，D：else

  ```mysql
  CREATE FUNCTION test_if(score INT) RETURNS CHAR
  BEGIN
  	IF score>=90 AND score<=100 THEN RETURN 'A';
  	ELSEIF score>=80 THEN RETURN 'B';
  	ELSEIF score>=60 THEN RETURN 'C';
  	ELSE RETURN 'D';
  	END IF;
  END $
  
  SELECT test_if(95) $
  ```

## 二、循环结构

- 分类：while、loop、repeat

- 循环控制：
  - iterate，类似于continue，继续，结束本次循环，继续下一次
  - leave，类似于break，跳出，结束当前所在的循环
- 三种循环都可以省略【标签：】，但如果循环中添加了循环控制语句（`leave`或`iterate`）则必须添加【标签：】

### 1. while循环

先判断后执行

```mysql
【标签：】 WHILE 循环条件 do
	循环体；
END WHILE 【标签】；
```

### 2. loop循环

可以用来模拟简单的死循环

```mysql
【标签：】 LOOP
	循环体；
END LOOP 【标签】；
```

### 3. repeat循环

先执行后判断，无条件至少执行一次

```mysql
【标签：】 REPEAT
	循环体；
UNTIL 结束循环的条件
END REPEAT 【标签】；
```

### 4. 案例

- 案例1：

  批量插入，向admin中插入指定数量的记录

  ```mysql
  CREATE PROCEDURE pro_while(IN insertCount INT)
  BEGIN
  	DECLARE i INT DEFAULT 1;
  	WHILE i<= insertCount DO
  		INSERT INTO admin(username,password)
  		VALUES(CONCAT('Rose',i),'password');
  		SET i=i+1;
  	END WHILE;
  END $
  
  CALL pro_while(100)$
  ```

- 案例2：添加`leave`语句

  批量插入，向admin中插入指定数量的记录，但是数量超过20时即停止

  ```mysql
  CREATE PROCEDURE test_while(IN insertCount INT)
  BEGIN
  	DECLARE i INT DEFAULT 1;
  	a: WHILE i<=insertCount DO
  		INSERT INTO admin(username,password)
  		VALUES(CONCAT('Jake',i),'password2');
  		IF i>=20 THEN LEAVE a; -- 【if i=20 then leave a】也可以
  		END IF;
  		SET i=i+1;
  	END WHILE a;
  END $
  
  CALL test_while(100) $
  ```

- 案例3：添加`iterate`语句

  批量插入，向admin中插入多条记录，只插入偶数次

  ```mysql
  CREATE PROCEDURE test_while(IN insertCount INT)
  BEGIN
  	DECLARE i INT DEFAULT 0;
  	a: WHILE i<=insertCount DO
  		SET i=i+1;
  		IF MOD(i,2)!=0 THEN ITERATE a; 
  		END IF;
  		INSERT INTO admin(username,password)
  		VALUES(CONCAT('Jake',i),'password2');
  	END WHILE a;
  END $
  
  CALL test_while(100) $
  ```

- 案例4

  向表中插入指定数量的随机的字符串

  ```mysql
  -- 先创建表格，两个字段：id、content
  CREATE TABLE stringcontent(
  	id INT PRIMARY KEY AUTO_INCREMENT,
  	content VARCHAR(20)
  );
  -- 创建存储过程
  CREATE PROCEDURE test_randStrInsert(IN insertCount INT)
  BEGIN
  	DECLARE i INT DEFAULT 1;
  	DECLARE str VARCHAR(26) DEFAULT 'qwertyuiopasdfghjklzxcvbnm';
  	DECLARE startIndex INT DEFAULT 1;
  	DECLARE len INT DEFAULT 1; 
  	WHILE i<= insertCount DO
  		SET startIndex=FLOOR(RAND()*26+1);
  		SET len=FLOOR(RAND()*(20-startIndex)+1);
  		-- 此处有问题，表格中content字段设置了最大长度为20
  		-- len的长度应 <=20，但15行中的算法并不正确
  		-- 个人感觉要用到 MIN()或 if()函数来限制len的长度
  		INSERT INTO stringContent(content)
  		VALUES (SUBSTR(str,startIndex,len));
  		SET i=i+1;
  	END WHILE;
  END $
  
  CALL test_randStrInsert(100) $
  ```

  
