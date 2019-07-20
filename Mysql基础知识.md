## Mysql基础知识

------

1.数据库基本概念

2.Mysql数据库软件

3.SQL

### 数据库的基本概念

```
1. 数据库的英文单词： DataBase 简称 ： DB
2. 什么数据库？
	* 用于存储和管理数据的仓库。

3. 数据库的特点：
	1. 持久化存储数据的。其实数据库就是一个文件系统
	2. 方便存储和管理数据
	3. 使用了统一的方式操作数据库 -- SQL
```

### MySQL数据库软件

------

```
配置
	* MySQL服务启动
		1. 手动。
		2. cmd--> services.msc 打开服务的窗口
		3. 使用管理员打开cmd
			* net start mysql : 启动mysql的服务
			* net stop mysql:关闭mysql服务
	* MySQL登录
		1. mysql -uroot -p密码
		2. mysql -hip -uroot -p连接目标的密码
		3. mysql --host=ip --user=root --password=连接目标的密码
	* MySQL退出
		1. exit
		2. quit

	* MySQL目录结构
		1. MySQL安装目录：basedir="D:/develop/MySQL/"
			* 配置文件 my.ini
		2. MySQL数据目录：datadir="C:/ProgramData/MySQL/MySQL Server 5.5/Data/"
			* 几个概念
				* 数据库：文件夹
				* 表：文件
				* 数据：数据
```

### SQL

------

```
1.什么是SQL？
	Structured Query Language：结构化查询语言
	其实就是定义了操作所有关系型数据库的规则。每一种数据库操作的方式存在不一样的地方，称为“方言”。
	
2.SQL通用语法
	1. SQL 语句可以单行或多行书写，以分号结尾。
	2. 可使用空格和缩进来增强语句的可读性。
	3. MySQL 数据库的 SQL 语句不区分大小写，关键字建议使用大写。
	4. 3 种注释
		* 单行注释: -- 注释内容 或 # 注释内容(mysql 特有) 
		* 多行注释: /* 注释 */
	
3. SQL分类
	1) DDL(Data Definition Language)数据定义语言
		用来定义数据库对象：数据库，表，列等。关键字：create, drop,alter 等
	2) DML(Data Manipulation Language)数据操作语言
		用来对数据库中表的数据进行增删改。关键字：insert, delete, update 等
	3) DQL(Data Query Language)数据查询语言
		用来查询数据库中表的记录(数据)。关键字：select, where 等
	4) DCL(Data Control Language)数据控制语言(了解)
		用来定义数据库的访问权限和安全级别，及创建用户。关键字：GRANT， REVOKE 等
```

#### DDL：操作数据库`表

```
1.创建数据库：CRUD
	1.C(create):创建
		创建数据库：
			create database 数据库名称;
		创建数据库，判断不存在，再创建：
			create database if not exists 数据库名称;
		创建数据库，并指定字符集
			create database 数据库名称 character set 字符集名;
	2.R(Retrieve):查询：
		查询所有数据库名称：
			show database;
		查询某个数据库的字符集：查询某个数据库的创建语句：
			show create database 数据库名称;
	3.U(Update):修改
		修改数据库的字符集
			alter database 数据库名称 character set 字符集名称;
	4.D(Delete):删除
		删除数据库
			drop database 数据库名称;
		判断数据库存在，存在再删除
			drop database if exists 数据库名称;
	5.使用数据库
		查询当前正在使用的数据库
			select database();
		使用数据库
			use 数据库名称;
```

```
2.操作表
	1.C(Create):创建
		1.语法：
			create table 表名(
				列表1 数据类型1，
				列表2 数据类型2，
				....
				列表n 数据类型n
			);
			数据库类型：
            	1.int 整数类型
            	2.double 小数类型
            	3.date 日期，只包含年月日,yyyy-MM-dd
            	4.datetime 日期，包含年月日时分秒 yyyy-MM-dd HH:mm:ss
            	5.timestamp 时间类型 包含年月日 yyyy-MM-dd HH:mm:ss
            	6.varchar:字符串
            复制表：
            	create table 表名 like 被复制的表名;
	2.R(Retrieve) 查询
		查询某个数据库中所有表名称
			show tables;
		查询表结构
			desc 表名;
	3.U(Update):修改：
		1.修改表名:
			alter table 表名 rname to 新的表名;
		2.修改表的字符集
			alter table 表名 character set 字符集名称;
		3.添加一列
			alter table 表名 add 列名 数据类型;
		4.修改列名称 类型
			alter table 表名 change 列名 新列 新数据类型;
			alter table 表名 modify 列名 新数据类型;
		5.删除列
			alter table 表名 drop 列名;
	4.D(Delete):删除：
		drop table 表名;
		drop table if exists 表名;
```

#### DML:增删改表中数据

```
1.添加数据
	语法
		insert into 表名(列名1,列名2,列名3,...列名n) values(值1,值2,值3,....,值n);
		insert into 表名 values (值1,值2,值3,....,值n);
2.删除数据
	语法:
		delete from 表名 [where 条件]
		如不加条件则删除表中所有数据
		1.delete from 表名;--有多少条数据就会执行多少次操作
		2.truncate table 表名;--先删除表，在创建一张一模一样的表
3.修改数据
	语法：
		update 表名 set 列名1=值1,列名2=值2,...列名n=值n,[where 条件];如果不加任何条件，则表中所有数据都会被修改
```

#### DQL:查询表中记录

```
select * from 表名;
1.语法
	select 
		字段列表
	from 
		表名列表
	where
		条件列表
	group by
		分组字段
	having 
		分组之后的条件
	order by
		排序
	limit 
	 	分页限定
```

```
2.基础查询
	1.多个字段的查询
		select 字段名1,字段名2,... from 表名;
		如查询所有字段，用*代替
	2.去除重复
		distinct
```

```sql
3.条件查询
	1.where子句后跟条件
	2.运算符
		>,<,<=,>=,=,<>
		between...and
		like:模糊查询
			占位符	；
				_:单个任意字符
				%:多个任意字符
		is null
		and 或 &&
		or 或 ||
		not 或 ！
	eg：
		查询年龄大于20岁
			select * from student where age > 20;
		查询年龄不等于20
			select * from student where age != 20;
			select *from student where age <> 20;
		查询年龄大于20小于等于30
			select *from student where age > 20 && age <= 30;
			select *from student where age > 20 and age <= 30;
			select *from student where age between 20 and 30;
		查询成绩为null
			select * from student where chengji is null;
		查询英语成绩不为null
		 select * from student where chengji is not null;
		 模糊查询eg:
		 	查询姓马的有哪些？ like
		 	select * from student where name like '马%';
		 	查询姓名第二个字是路的人
		 	select * from student where name like "_路%";
		 	查询姓名是三个字的人
		 	select * from student where name like "___";
		 	查询姓名中包含娜的人
		 	select * from student where name like "%娜%";
```

```
4.排序查询
	语法:order by 子句
		order by 排序字段1 排序方式1，排序字段2 排序方式2
		排序方式：
		ASC:升序，默认的。
		DESC:降序
5.聚合函数：将一列数据作为一个整体，进行纵向的计算。
	1.count：计算个数
	2.max：计算最大值
	3.min: 计算最小值
	4.sum：计算和
	5.avg：计算平均值
6.分组查询：
	1.语法:group by 分组字段;
	2注意：1. 分组之后查询的字段：分组字段、聚合函数
		2. where 和 having 的区别？
			1. where 在分组之前进行限定，如果不满足条件，则不参与分组。having在分组之后进行限定，如果不满足结果，则不会被查询出来
			2. where 后不可以跟聚合函数，having可以进行聚合函数的判断。

		-- 按照性别分组。分别查询男、女同学的平均分

		SELECT sex , AVG(math) FROM student GROUP BY sex;
		
		-- 按照性别分组。分别查询男、女同学的平均分,人数
		
		SELECT sex , AVG(math),COUNT(id) FROM student GROUP BY sex;
		
		--  按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组
		SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex;
		
		--  按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组,分组之后。人数要大于2个人
		SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex HAVING COUNT(id) > 2;
		
		SELECT sex , AVG(math),COUNT(id) 人数 FROM student WHERE math > 70 GROUP BY sex HAVING 人数 > 2;
		
```

```
7.分页查询
	1.语法：limit 开始的索引，每页查询的条数
	2.公式：开始的索引 = （当前的页码 - 1) * 每页显示的条数
		eg：每页显示三条记录
		select * from student limit 0,3;
		select * from student limit 3,3;\
		select * from student limit 6,3;
		limit 是mysql一个方言
```

