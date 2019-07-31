SQL语言学习——DDL部分
-------

——作者：Su

- 库的管理

  创建（create）、修改（alter）、删除（drop）

- 表的管理

  创建（create）、修改（alter）、删除（drop）

## 一、库的管理

### 1. 库的创建

`create database 【if not exists】 库名 【character set 字符集名】；`

在【C:\ProgramData\MySQL\MySQL Server 8.0\Data】路径下可以看到本机数据库的列表

如果没有数据库则创建，提高容错率；

```mysql
CREATE DATABASE IF NOT EXISTS books;
```

### 2. 库的修改

- ***库名的修改，此方法不稳定，不推荐***

  *没有修改库名的SQL语句，`RENAME DATABASE 库名 TO 新库名`语句已不可用，但是可以通过如下方法修改库名：*

  - *停止服务*
  - *在【C:\ProgramData\MySQL\MySQL Server 8.0\Data】路径下修改数据库对应文件夹的名字*
  - *重启服务*

- 更改库的字符集

  ```mysql
  alter database books character set gbk;
  ```

- 库的删除

  ```mysql
  drop database if exists books;
  ```

## 二、表的管理

### 1. 表的创建

```mysql
create table 表名(
	列名 列的类型 【（长度）约束】,
	列名 列的类型 【（长度）约束】,
	……
	列名 列的类型 【（长度）约束】
);
```

- 案例1：创建表book

  ```mysql
  CREATE TABLE book(
  	id int,
  	bName VARCHAR(20),
  	price DOUBLE,
  	authorID INT,
  	publishDate DATETIME
  );
  ```

- 案例2：创建表author

  ```mysql
  CREATE TABLE author(
  	id INT,
  	au_name VARCHAR(20),
  	nation VARCHAR(10)
  );
  ```

### 2. 表的修改

```mysql
alter table 表名 add|drop|modify|change column 列名【列类型 约束】;
```

- 列名的修改

  ```mysql
  ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;
  ```

- 列的类型或约束的修改

  ```mysql
  ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;
  ```

- 列的添加

  `ALTER TABLE 表名 ADD 列名 类型 【first|after 字段名】;`

  【first | after 字段名】用来设置添加的列的位置

  ```mysql
  ALTER TABLE author ADD annual DOUBLE;
  ```

- 列的删除

  ```mysql
  ALTER TABLE author DROP annual;
  ```

- 表名的修改

  ```mysql
  ALTER TABLE author RENAME TO book_author;
  ```

### 3. 表的删除

```mysql
DROP TABLE book_author;
```

```mysql
DROP TABLE IF EXISTS book_author;
```

- 一般通用写法

  ```mysql
DROP DATABASE IF EXISTS 旧库名;
CREATE DATABASE 新库名;
DROP TABLE IF EXISTS 旧表名;
CREATE TABLE 表名();
  ```

### 4. 表的复制

- 仅复制表的结构

  ```mysql
  CREATE TABLE copy LIKE author;
  ```

  *copy为新表名*

- 复制表的结构+数据

  ```mysql
  CREATE TABLE copy2
  SELECT * FROM author;
  ```

  *copy2为新表名*

- 仅复制部分数据

  ```mysql
  CREATE TABLE copy3
  SELECT id,au_name
  FROM author
  where nation='中国';
  ```

  *copy2为新表名*

- 仅复制部分结构

  ```mysql
  CREATE TABLE copy4
  SELECT id,au_name
  FROM author
  where 0;
  ```

  `where 0`限制了没有任何条目满足条件

## 三、常见的数据类型

- 数值型
  - 整型
  - 小数：
    - 定点数
    - 浮点数
- 字符型：
  - 较短的文本：char、varchar
  - 较长的文本：text、blob（较长的二进制数据）

- 日期型

### 1. 整型

tinyint（1字节）、smallint（2字节）、mediumint（3字节）、int/integer（4字节）、bigint（8字节）

![1558946199641](md image/SQL语言学习——DDL部分/1558946199641.png)

- 特点：
  1. 如果不设置有符号或者无符号，默认有符号，如果像设置无符号，需要添加unsigned关键字
  2. 如果插入的数值超过了整型的范围，会报out of range异常，并且插入临界值
  3. 若不设置长度，会有默认长度
  4. 长度代表了显示的最大宽度，如果不够会用0在左边填充，但必须搭配`zerofill`使用

- 如何设置无符号和有符号

  使用`UNSIGNED`

  ```mysql
  CREATE TABLE tab_int(
  	t1 INT(7) ZEROFILL,
  	t2 INT UNSIGNED
  );
  ```

### 2. 小数

![1558946474960](md image/SQL语言学习——DDL部分/1558946474960.png)

- 浮点型
  - float(M,D)
  - double(M,D)
- 定点型
  - dec(M,D)
  - decimal(M,D)

- 特点

  1. M：整数部位+小数部位总长度

     D：小数部位长度

     如果超过范围，则插入临界值

  2. M和D都可以省略

     但是decimal，则M默认为10，D默认为0

     float、double，则会根据插入数值的精度来决定精度

  3. 定点型的精确的较高，如果要求插入数值的精度较高，如货币运算等，则考虑使用定点型

- **原则：所选择的类型越简单越好，能保存数值的类型越小越好**

### 3. 字符型

- 较短的文本：
  - char
  - varchar
  - binary，varbinary：保存较短的二进制
  - enum：保存枚举
  - set：保存集合
- 较长的文本：
  - text
  - blob（较大的二进制）

![1558948374645](md image/SQL语言学习——DDL部分/1558948374645.png)

**M为字符数，不是字节数**

char比较耗费空间，varchar可根据数据的长度改变空间占用量，但varchar效率较低

|         | 写法       | M的含义                           | 特点           | 空间的耗费 | 效率 |
| ------- | ---------- | --------------------------------- | -------------- | ---------- | ---- |
| char    | char(M)    | 最大字符数，**可以省略，默认为1** | 固定长度的字符 | 比较耗费   | 高   |
| varchar | varchar(M) | 最大字符数，不可省略              | 可变长度的字符 | 比较节省   | 低   |

- ENUM枚举型

  ```mysql
  CREATE TABLE tab_char(
  	c1 ENUM('a','b','c')
  );
  INSERT INTO tab_char VALUES('a');
  INSERT INTO tab_char VALUES('b');
  INSERT INTO tab_char VALUES('c');
  INSERT INTO tab_char VALUES('A'); -- 插入小写a
  INSERT INTO tab_char VALUES('m'); -- 报错
  ```

- SET集合型

  ```mysql
  CREATE TABLE tab_set(
  	s1 SET('a','b','c','d')
  );
  INSERT INTO tab_set VALUES('a');
  INSERT INTO tab_set VALUES('a,b'); -- 注意：不是【'a','b'】
  INSERT INTO tab_set VALUES('a,b,c');
  ```

### 4.  日期型

- date：只保存日期

- time：只保存时间

- year：只保存年

- datetime：保存日期+时间，不受时区影响

- timestamp：保存日期+时间，受时区影响

![1559005476282](md image/SQL语言学习——DDL部分/1559005476282.png)

## 四、常见约束

一种限制，用于限制表中数据，以保证表中数据的准确和可靠性

**六大约束：**

- NOT NULL：非空，用于保证该字段的值不能为空，如姓名学号等
- DEFAULT：默认，用于保证该字段有默认值
- PRIMARY KEY：主键，用于保证该字段的值有唯一性，并且非空，如学号工号等
- UNIQUE：唯一，用于保证该字段的值具有唯一性，可以为空
- CHECK：检查约束【mysql中不支持】，比如年龄、性别，限制数据符合真实性
- FOREIGN KEY：外键，用于限制两个表的关系，用于保证该字段的值必须来自主表的关联列的值，在从表添加外键约束，用于引用主表中某列的值

**添加约束的设计：**

- 创建表时
- 修改表时

**约束的分类：**

- 列级约束
  - 六大约束语法上都支持，但外键约束没有效果
- 表级约束
  - 除了非空、默认，其他都支持

### 1. 创建表时添加约束

#### 1）添加列级约束

直接在字段名和类型后面追加约束类型即可，一个字段后面可以添加多个约束，空格连接

只支持：默认、非空、主键、唯一

```mysql
CREATE TABLE stuinfo(
	id INT PRIMARY KEY,
	stuName VARCHAR(20) NOT NULL,
	gender CHAR(1) CHECK(gender='男' OR gender='女'),
	seat INT UNIQUE,
	age INT DEFAULT 18,
	majorId INT REFERENCES major(id)
);

CREATE TABLE major(
	id INT PRIMARY KEY,
	majorName VARCHAR(20)
);
```

#### 2）添加表级约束

在各个字段的最下面

`【constraint 约束名】 约束类型（字段名）`

```mysql
CREATE TABLE major(
	id INT PRIMARY KEY,
	majorName VARCHAR(20)
);

CREATE TABLE stuinfo(
	id INT,
	stuName VARCHAR(20) ,
	gender CHAR(1) ,
	seat INT,
	age INT,
	majorId INT,

	CONSTRAINT pk PRIMARY KEY(id),
	CONSTRAINT uq UNIQUE(seat),
	CONSTRAINT ck CHECK(gender='男' OR gender='女'),
	CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorId) REFERENCES major(id)
);
```

#### 3）总结

通用写法

```mysql
CREATE TABLE IF NOT EXISTS stuinfo(
	id INT PRIMARY KEY,
    stuName VARCHAR(20) NOT NULL,
	gender CHAR(1),
	seat INT UNIQUE,
	age INT DEFAULT 18,
	majorId INT,
    
    CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorId) REFERENCES major(id)
)
```

- **主键和唯一键都可以组合**

  `PRIMARY KEY(first_name,last_name)`

  `UNIQUE (first_name,last_name)`

- 外键：
  1. 要求在从表设置外键关系
  2. 从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求
  3. 主表的关联列必须是一个key，一般是主键或唯一键
  4. 插入数据时，先插入主表，再插入从表，删除数据时，先删除从表，再删除主表

### 2. 修改表时添加约束

**列级约束**：`alter table 表名 modify column 字段名 字段类型 新约束;`

**表级约束**：`alter table 标名 [constrain 约束名] 约束类型(字段名) [外键的引用];`

- 添加非空约束

  ```mysql
  ALTER TABLE stuinfo MODIFY stuname VARCHAR(20) NOT NULL;
  ALTER TABLE stuinfo MODIFY stuname VARCHAR(20); -- 去掉非空约束
  ```

- 添加默认约束

  ```mysql
  ALTER TABLE stuinfo MODIFY age INT DEFAULT 18;
  ```

- 添加主键

  ```mysql
  ALTER TABLE stuinfo MODIFY id INT PRIMARY KEY;
  ```

  ```mysql
  ALTER TABLE stuinfo ADD PRIMARY KEY(id);
  ```

- 添加唯一

  ```mysql
  ALTER TABLE stuinfo MODIFY seat INT UNIQUE;
  ```

  ```mysql
  ALTER TABLE stuinfo ADD UNIQUE(seat);
  ```

- 添加外键

  ```mysql
  ALTER TABLE stuinfo ADD CONSTRAINT fk_stuinfo_major 
  	FOREIGN KEY(majorId) REFERENCES major(id);
  ```

### 3. 修改表时删除约束

```mysql
ALTER TABLE stuinfo MODIFY stuname VARCHAR(20) NULL;
ALTER TABLE stuinfo MODIFY age INT;
ALTER TABLE stuinfo DROP PRIMARY KEY;
ALTER TABLE stuinfo DROP INDEX seat; -- 删除唯一键,需要写名字
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;
```

## 五、标识列

即自增长列

1. 标识列不一定和主键搭配，但要求是一个**key**
2. 一个表至多有一个标识列
3. 标识列类型只能是数值型
4. 标识列可以通过`SET auto_increment_increment=3;`设置步长
5. 可以通过手动插入值,设置起始值

### 1. 创建表时设置标识列

加关键字`AUTO_INCREMENT`

```mysql
CREATE TABLE tab_identity(
	id int PRIMARY KEY auto_increment,
	name VARCHAR(20)
);
```

插入数据时，标识列可以写`null`，也可以设置为指定值

```mysql
INSERT INTO tab_identity VALUES(NULL,'hofa');
INSERT INTO tab_identity(name) VALUES('hofa');
INSERT INTO tab_identity VALUES(5,'hofa');
```

### 2. 修改表时设置标识列

`ALTER TABLE tab_identity MODIFY id INT PRIMARY KEY ANTO_INCREMENT;`

### 3. 修改表时删除标识列

`ALTER TABLE tab_identity MODIFY id INT;`

