### sql优化(mysql)

#### 1.mysql

##### 数据库存放目录：

```
	ps -ef|grep mysql  可以看到：
		数据库目录：     datadir=/var/lib/mysql 
		pid文件目录： --pid-file=/var/lib/mysql/bigdata01.pid

		MySQL核心目录：
			/var/lib/mysql :mysql 安装目录
			/usr/share/mysql:  配置文件
			/usr/bin：命令目录（mysqladmin、mysqldump等）
			/etc/init.d/mysql启停脚本
```

##### MySQL配置文件

```
my-huge.cnf	高端服务器  1-2G内存
			 my-large.cnf   中等规模
			 my-medium.cnf  一般
			 my-small.cnf   较小
			但是，以上配置文件mysql默认不能识别，默认只能识别 /etc/my.cnf
			采用 my-huge.cnf ：
			cp /usr/share/mysql/my-huge.cnf /etc/my.cnf
			注意：mysql5.5默认配置文件/etc/my.cnf；Mysql5.6 默认配置文件/etc/mysql-default.cnf
```

##### 默认端口3306

##### mysql字符编码：

```
sql  :  show variables like '%char%' ;
		可以发现部分编码是 latin,需要统一设置为utf-8
		设置编码：
		vi /etc/my.cnf:
		[mysql]
		default-character-set=utf8
		[client]
		default-character-set=utf8
		
		[mysqld]
		character_set_server=utf8
		character_set_client=utf8
		collation_server=utf8_general_ci
		重启Mysql:  service mysql restart
		sql  :  show variables like '%char%' ;
		注意事项：修改编码 只对“之后”创建的数据库生效，因此 我们建议 在mysql安装完毕后，第一时间 统一编码。
		mysql:清屏    ctrl+L    , system clear
```

#### 2.原理

##### MYSQL逻辑分层 ：

连接层 服务层 引擎层 存储层

![1257218-20180930171620917-387740964](C:\Users\DH\Desktop\1257218-20180930171620917-387740964.png)

连接层：连接与线程处理，这一层并不是MySQL独有，一般的基于C/S架构的都有类似组件，比如连接处理、授权认证、安全等。

服务层：包括缓存查询、解析器、优化器，这一部分是MySQL核心功能，包括解析、优化SQL语句，查询缓存目录，内置函数（日期、时间、加密等函数）的实现。

引擎层：负责数据存储，存储引擎的不同，存储方式、数据格式、提取方式等都不相同，这一部分也是很大影响数据存储与提取的性能的；对存储层的抽象。

存储层：存储数据，文件系统。

##### 储存引擎

存储引擎是数据库底层软件组织，数据库管理系统（DBMS）使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以 获得特定的功能。现在许多不同的数据库管理系统都支持多种不同的数据引擎；特别提示：存储引擎针对于数据表。

```
InnoDB(默认) ：事务优先 （适合高并发操作；行锁）
MyISAM ：性能优先  （表锁）
查询数据库引擎：  支持哪些引擎？ show engines ;
			查看当前使用的引擎   show variables like '%storage_engine%' ;
```

###### 指定数据库对象的引擎：

```
create table tb(
		id int(4) auto_increment ,
		name varchar(5),
		dept varchar(5) ,
		primary key(id)		
	)ENGINE=MyISAM AUTO_INCREMENT=1
	 DEFAULT CHARSET=utf8   ;
```

#### 3.SQL优化

##### 原因：

```
性能低、执行时间太长、等待时间太长、SQL语句欠佳（连接查询）、索引失效、服务器参数设置不合理（缓冲、线程数）
```

###### 编写过程：

```
select dinstinct  ..from  ..join ..on ..where ..group by ...having ..order by ..limit ..
```

###### 解析过程：

```
from .. on.. join ..where ..group by ....having ...select dinstinct ..order by limit ...
```

##### SQL优化， 主要就是 在优化索引

```
索引： 相当于书的目录
索引： index是帮助MYSQL高效获取数据的数据结构。索引是数据结构（树：B树(默认)、Hash树...）
```

###### 索引的弊端：

```
1.索引本身很大， 可以存放在内存/硬盘（通常为 硬盘）
			2.索引不是所有情况均适用： a.少量数据  b.频繁更新的字段   c.很少使用的字段
			3.索引会降低增删改的效率（增删改查）
```

###### 优势：

```
1提高查询效率（降低IO使用率）
2.降低CPU使用率 （...order by age desc,因为 B树索引 本身就是一个 好排序的结构，因此在排序时可以直接使用）
```

#### 4.索引

##### 分类

```
主键索引：不能重复。id不能是null
唯一索引：不能重复。id可以是null
单值索引： 单列，age ;一个表可以多个单值索引,name。
复合索引：多个列构成的索引 （相当于二级目录：z: zhao）  (name,age)   (a,b,c,d,...,n)
```

##### 创建索引：

```
方式一：
		create 索引类型  索引名  on 表(字段)
		单值：
		create index   dept_index on  tb(dept);
		唯一：
		create unique index  name_index on tb(name) ;
		复合索引
		create index dept_name_index on tb(dept,name);
```

```
方式二：alter table 表名 索引类型  索引名（字段）
		单值：
		alter table tb add index dept_index(dept) ;
		唯一：
		alter table tb add unique index name_index(name);
		复合索引
		alter table tb add index dept_name_index(dept,name);
		注意：如果一个字段是primary key，则改字段默认就是 主键索引	
```

##### 删除索引：

```
drop index 索引名 on 表名 ;
drop index name_index on tb ;
```

##### 查询索引：

```
show index from 表名 ;
show index from 表名 \G
```

#### 5.SQL性能问题

##### 分析SQL的执行计划  :

explain   ，可以模拟SQL优化器执行SQL语句，从而让开发人员 知道自己编写的SQL状况

##### MySQL查询优化其会干扰我们的优化

##### 查询执行计划：  

```
explain +SQL语句 explain  select  * from tb ;
```

```
id : 编号				
 select_type ：查询类型
 table ：表
 type   ：类型
 possible_keys ：预测用到的索引 
 key  ：实际使用的索引
 key_len ：实际使用索引的长度     
 ref  :表之间的引用
 rows ：通过索引查询到的数据量 
 Extra     :额外的信息
```

准备数据

```
create table course
(
cid int(3),
cname varchar(20),
tid int(3)
);
create table teacher
(
tid int(3),
tname varchar(20),
tcid int(3)
);

create table teacherCard
(
tcid int(3),
tcdesc varchar(200)
);


insert into course values(1,'java',1);
insert into course values(2,'html',1);
insert into course values(3,'sql',2);
insert into course values(4,'web',3);

insert into teacher values(1,'tz',1);
insert into teacher values(2,'tw',2);
insert into teacher values(3,'tl',3);

insert into teacherCard values(1,'tzdesc') ;
insert into teacherCard values(2,'twdesc') ;
insert into teacherCard values(3,'tldesc') ;
```

查询课程编号为2  或 教师证编号为3  的老师信息
explain +sql:

##### (1)id: id值相同，从上往下 顺序执行。

```
查询教授SQL课程的老师的描述（desc）
explain select tc.tcdesc from teacherCard tc,course c,teacher t where c.tid = t.tid
and t.tcid = tc.tcid and c.cname = 'sql' ;
将以上 多表查询 转为子查询形式：

explain select tc.tcdesc from teacherCard tc where tc.tcid = 
(select t.tcid from teacher t where  t.tid =  
	(select c.tid from course c where c.cname = 'sql')
);

子查询+多表： 
explain select t.tname ,tc.tcdesc from teacher t,teacherCard tc where t.tcid= tc.tcid
and t.tid = (select c.tid from course c where cname = 'sql') ;

id值有相同，又有不同： id值越大越优先；id值相同，从上往下 顺序执行
```

##### (2)select_type:查询类型

```
PRIMARY:包含子查询SQL中的 主查询 （最外层）
SUBQUERY：包含子查询SQL中的 子查询 （非最外层）
simple:简单查询（不包含子查询、union）
derived:衍生查询(使用到了临时表)
a.在from子查询中只有一张表
		explain select  cr.cname from ( select * from course where tid in (1,2) ) cr ;

b.在from子查询中， 如果有table1 union table2 ，则table1 就是derived,table2就是union
		explain select  cr.cname from ( select * from course where tid = 1  union select * from course where tid = 2 ) cr ;
union:上例
union result :告知开发人员，那些表之间存在union查询
```

```
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
```

##### (3)type:索引类型、类型

```
system>const>eq_ref>ref>range>index>all   ，要对type进行优化的前提：有索引

其中：system,const只是理想情况；实际能达到 ref>range

system（忽略）: 只有一条数据的系统表 ；或 衍生表只有一条数据的主查询
insert into test01 values(1,'a') ;
commit;
增加索引
alter table test01 add constraint tid_pk primary key(tid) ;
explain select * from (select * from test01 )t where tid =1 ;

const:仅仅能查到一条数据的SQL ,用于Primary key 或unique索引  （类型 与索引类型有关）
explain select tid from test01 where tid =1 ;
alter table test01 drop primary key ;
create index test01_index on test01(tid) ;

eq_ref:唯一性索引：对于每个索引键的查询，返回匹配唯一行数据（有且只有1个，不能多 、不能0）
select ... from ..where name = ... .常见于唯一索引 和主键索引。

 alter table teacherCard add constraint pk_tcid primary key(tcid);
alter table teacher add constraint uk_tcid unique index(tcid) ;


explain select t.tcid from teacher t,teacherCard tc where t.tcid = tc.tcid ;

以上SQL，用到的索引是 t.tcid,即teacher表中的tcid字段；
如果teacher表的数据个数 和 连接查询的数据个数一致（都是3条数据），则有可能满足eq_ref级别；否则无法满足。


ref：非唯一性索引，对于每个索引键的查询，返回匹配的所有行（0，多）
准备数据：
 insert into teacher values(4,'tz',4) ;
 insert into teacherCard values(4,'tz222');

测试：
alter table teacher add index index_name (tname) ;
explain select * from teacher 	where tname = 'tz';

range：检索指定范围的行 ,where后面是一个范围查询(between   ,> < >=,     特殊:in有时候会失效 ，从而转为 无索引all)
alter table teacher add index tid_index (tid) ;
explain select t.* from teacher t where t.tid in (1,2) ;
explain select t.* from teacher t where t.tid <3 ;

index：查询全部索引中数据
explain select tid from teacher ; --tid 是索引， 只需要扫描索引表，不需要所有表中的所有数据

all：查询全部表中的数据
explain select cid from course ;  --cid不是索引，需要全表所有，即需要所有表中的所有数据
system/const: 结果只有一条数据
eq_ref:结果多条；但是每条数据是唯一的 ；
ref：结果多条；但是每条数据是是0或多条 ；
```

##### （4）possible_keys ：可能用到的索引，是一种预测，不准。

```
alter table  course add index cname_index (cname);

explain select t.tname ,tc.tcdesc from teacher t,teacherCard tc
 where t.tcid= tc.tcid
and t.tid = (select c.tid from course c where cname = 'sql') ;


如果 possible_key/key是NULL，则说明没用索引


explain select tc.tcdesc from teacherCard tc,course c,teacher t where c.tid = t.tid
and t.tcid = tc.tcid and c.cname = 'sql' ;
```

##### （5） key ：实际使用到的索引

##### （6）key_len ：索引的长度 ;

```
作用：用于判断复合索引是否被完全使用  （a,b,c）。
create table test_kl
(
	name char(20) not null default ''
);
alter table test_kl add index index_name(name) ;
explain select * from test_kl where name ='' ;   -- key_len :60
在utf8：1个字符站3个字节  

alter table test_kl add column name1 char(20) ;  --name1可以为null

alter table test_kl add index index_name1(name1) ;
explain select * from test_kl where name1 ='' ; 
--如果索引字段可以为Null,则会使用1个字节用于标识。

drop index index_name on test_kl ;
drop index index_name1 on test_kl ;

增加一个复合索引 
alter table test_kl add index name_name1_index (name,name1) ; 

explain select * from test_kl where name1 = '' ; --121
explain select * from test_kl where name = '' ; --60


varchar(20)
alter table test_kl add column name2 varchar(20) ; --可以为Null 
alter table test_kl add index name2_index (name2) ;

explain select * from test_kl where name2 = '' ;  --63
20*3=60 +  1(null)  +2(用2个字节 标识可变长度)  =63

utf8:1个字符3个字节
gbk:1个字符2个字节
latin:1个字符1个字节
```

##### (7) ref : 注意与type中的ref值区分。

```
作用： 指明当前表所 参照的 字段。
		select ....where a.c = b.x ;(其中b.x可以是常量，const)

alter table course  add index tid_index (tid) ;

	explain select * from course c,teacher t where c.tid = t.tid  and t.tname ='tw' ;
```

##### (8)rows: 被索引优化查询的 数据个数 (实际通过索引而查询到的 数据个数)

```
explain select * from course c,teacher t  where c.tid = t.tid
	and t.tname = 'tz' ;
```

##### （9）Extra：

```
(i).using filesort ： 性能消耗大；需要“额外”的一次排序（查询）  。常见于 order by 语句中。
排序：先查询

10个人 根据年龄排序。

	
create table test02
(
	a1 char(3),
	a2 char(3),
	a3 char(3),
	index idx_a1(a1),
	index idx_a2(a2),
	index idx_a3(a3)
);

explain select * from test02 where a1 ='' order by a1 ;

a1:姓名  a2：年龄


explain select * from test02 where a1 ='' order by a2 ; --using filesort
小结：对于单索引， 如果排序和查找是同一个字段，则不会出现using filesort；如果排序和查找不是同一个字段，则会出现using filesort；
	避免： where哪些字段，就order by那些字段2


复合索引：不能跨列（最佳左前缀）
drop index idx_a1 on test02;
drop index idx_a2 on test02;
drop index idx_a3 on test02;

alter table test02 add index idx_a1_a2_a3 (a1,a2,a3) ;
explain select *from test02 where a1='' order by a3 ;  --using filesort
explain select *from test02 where a2='' order by a3 ; --using filesort
explain select *from test02 where a1='' order by a2 ;
explain select *from test02 where a2='' order by a1 ; --using filesort
	小结：避免： where和order by 按照复合索引的顺序使用，不要跨列或无序使用。


	(ii). using temporary:性能损耗大 ，用到了临时表。一般出现在group by 语句中。
	explain select a1 from test02 where a1 in ('1','2','3') group by a1 ;
	explain select a1 from test02 where a1 in ('1','2','3') group by a2 ; --using temporary
	避免：查询那些列，就根据那些列 group by .

	(iii). using index :性能提升; 索引覆盖（覆盖索引）。原因：不读取原文件，只从索引文件中获取数据 （不需要回表查询）
		只要使用到的列 全部都在索引中，就是索引覆盖using index

	例如：test02表中有一个复合索引(a1,a2,a3)
		explain select a1,a2 from test02 where a1='' or a2= '' ; --using index   
		
		drop index idx_a1_a2_a3 on test02;

		alter table test02 add index idx_a1_a2(a1,a2) ;
		explain select a1,a3 from test02 where a1='' or a3= '' ;

		
		如果用到了索引覆盖(using index时)，会对 possible_keys和key造成影响：
		a.如果没有where，则索引只出现在key中；
		b.如果有where，则索引 出现在key和possible_keys中。

		explain select a1,a2 from test02 where a1='' or a2= '' ;
		explain select a1,a2 from test02  ;

	(iii).using where （需要回表查询）
		假设age是索引列
		但查询语句select age,name from ...where age =...,此语句中必须回原表查Name，因此会显示using where.
		
	explain select a1,a3 from test02 where a3 = '' ; --a3需要回原表查询
	

	(iv). impossible where ： where子句永远为false
		explain select * from test02 where a1='x' and a1='y'  ;


```

#### 6.优化案例

单表优化、两表优化、三表优化

##### （1）单表优化

```
create table book
(
	bid int(4) primary key,
	name varchar(20) not null,
	authorid int(4) not null,
	publicid int(4) not null,
	typeid int(4) not null 
);

insert into book values(1,'tjava',1,1,2) ;
insert into book values(2,'tc',2,1,2) ;
insert into book values(3,'wx',3,2,1) ;
insert into book values(4,'math',4,2,3) ;	
commit;	

	查询authorid=1且 typeid为2或3的	bid
	explain select bid from book where typeid in(2,3) and authorid=1  order by typeid desc ;

	(a,b,c)
	(a,b)

	优化：加索引
	alter table book add index idx_bta (bid,typeid,authorid);

	索引一旦进行 升级优化，需要将之前废弃的索引删掉，防止干扰。
	drop index idx_bta on book;

	根据SQL实际解析的顺序，调整索引的顺序：
	alter table book add index idx_tab (typeid,authorid,bid); --虽然可以回表查询bid，但是将bid放到索引中 可以提升使用using index ;
	
	再次优化（之前是index级别）：思路。因为范围查询in有时会实现，因此交换 索引的顺序，将typeid in(2,3) 放到最后。
	drop index idx_tab on book;
	alter table book add index idx_atb (authorid,typeid,bid);
	explain select bid from book where  authorid=1 and  typeid in(2,3) order by typeid desc ;

	
	--小结：	a.最佳做前缀，保持索引的定义和使用的顺序一致性  b.索引需要逐步优化  c.将含In的范围查询 放到where条件的最后，防止失效。

	本例中同时出现了Using where（需要回原表）; Using index（不需要回原表）：原因，where  authorid=1 and  typeid in(2,3)中authorid在索引(authorid,typeid,bid)中，因此不需要回原表（直接在索引表中能查到）；而typeid虽然也在索引(authorid,typeid,bid)中，但是含in的范围查询已经使该typeid索引失效，因此相当于没有typeid这个索引，所以需要回原表（using where）；
	例如以下没有了In，则不会出现using where
	explain select bid from book where  authorid=1 and  typeid =3 order by typeid desc ;

	还可以通过key_len证明In可以使索引失效。

```

##### （2）两表优化

```
create table teacher2
(
	tid int(4) primary key,
	cid int(4) not null
);

insert into teacher2 values(1,2);
insert into teacher2 values(2,1);
insert into teacher2 values(3,3);

create table course2
(
	cid int(4) ,
	cname varchar(20)
);

insert into course2 values(1,'java');
insert into course2 values(2,'python');
insert into course2 values(3,'kotlin');
commit;

左连接：
	explain select *from teacher2 t left outer join course2 c
	on t.cid=c.cid where c.cname='java';
	  

	索引往哪张表加？   -小表驱动大表  
		          -索引建立经常使用的字段上 （本题 t.cid=c.cid可知，t.cid字段使用频繁，因此给该字段加索引） [一般情况对于左外连接，给左表加索引；右外连接，给右表加索引]
	小表：10
	大表：300
	where   小表.x 10 = 大表.y 300;  --循环了几次？10
		
		大表.y 300=小表.x 10	--循环了300次


	小表:10
	大表:300

	select ...where 小表.x10=大表.x300 ;
	for(int i=0;i<小表.length10;i++)
	{
		for(int j=0;j<大表.length300;j++)
		{
			...
		}
	}
	

	select ...where 大表.x300=小表.x10 ;
	for(int i=0;i<大表.length300;i++)
	{
		for(int j=0;j<小表.length10;j++)
		{
			...
		}
	}
	
--以上2个FOR循环，最终都会循环3000次；但是 对于双层循环来说：一般建议 将数据小的循环 放外层；数据大的循环放内存。



	--当编写 ..on t.cid=c.cid 时，将数据量小的表 放左边（假设此时t表数据量小）
	
	alter table teacher2 add index index_teacher2_cid(cid) ;
	alter table course2 add index index_course2_cname(cname);


	Using join buffer:extra中的一个选项，作用：Mysql引擎使用了 连接缓存。
```

##### （3）三张表优化A B C

```
a.小表驱动大表  b.索引建立在经常查询的字段上


示例：
create table test03
(
  a1 int(4) not null,
  a2 int(4) not null,
  a3 int(4) not null,
  a4 int(4) not null
);
alter table test03 add index idx_a1_a2_a3_4(a1,a2,a3,a4) ;

	
	explain select a1,a2,a3,a4 from test03 where a1=1 and a2=2 and a3=3 and a4 =4 ; --推荐写法，因为 索引的使用顺序（where后面的顺序） 和 复合索引的顺序一致

	explain select a1,a2,a3,a4 from test03 where a4=1 and a3=2 and a2=3 and a1 =4 ; --虽然编写的顺序 和索引顺序不一致，但是 sql在真正执行前 经过了SQL优化器的调整，结果与上条SQL是一致的。
	--以上 2个SQL，使用了 全部的复合索引

	explain select a1,a2,a3,a4 from test03 where a1=1 and a2=2 and a4=4 order by a3; 
	--以上SQL用到了a1 a2两个索引，该两个字段 不需要回表查询using index ;而a4因为跨列使用，造成了该索引失效，需要回表查询 因此是using where；以上可以通过 key_len进行验证

	explain select a1,a2,a3,a4 from test03 where a1=1 and a4=4 order by a3; 
	--以上SQL出现了 using filesort(文件内排序，“多了一次额外的查找/排序”) ：不要跨列使用( where和order by 拼起来，不要跨列使用)


	explain select a1,a2,a3,a4 from test03 where a1=1 and a4=4 order by a2 , a3; --不会using filesort

	
	--总结：i.如果 (a,b,c,d)复合索引  和使用的顺序全部一致(且不跨列使用)，则复合索引全部使用。如果部分一致(且不跨列使用)，则使用部分索引。
	select a,c where  a = and b= and d= 
		ii.where和order by 拼起来，不要跨列使用 

	
	using temporary:需要额外再多使用一张表. 一般出现在group by语句中；已经有表了，但不适用，必须再来一张表。
解析过程：			
from .. on.. join ..where ..group by ....having ...select dinstinct ..order by limit ...
	a.
		explain select * from test03 where a2=2 and a4=4 group by a2,a4 ;--没有using temporary
	b.
		explain select * from test03 where a2=2 and a4=4 group by a3 ;
```

#### 7.避免索引失效的一些原则 

##### （1）复合索引

```
a.复合索引，不要跨列或无序使用（最佳左前缀）
	(a,b,c)  
	b.复合索引，尽量使用全索引匹配
	(a,b,c)  
```

（2）不要在索引上进行任何操作（计算、函数、类型转换），否则索引失效

```
select ..where A.x = .. ;  --假设A.x是索引
		不要：select ..where A.x*3 = .. ;
		explain select * from book where authorid = 1 and typeid = 2 ;--用到了at2个索引
		explain select * from book where authorid = 1 and typeid*2 = 2 ;--用到了a1个索引
		explain select * from book where authorid*2 = 1 and typeid*2 = 2 ;----用到了0个索引
		explain select * from book where authorid*2 = 1 and typeid = 2 ;----用到了0个索引,原因：对于复合索引，如果左边失效，右侧全部失效。(a,b,c)，例如如果 b失效，则b c同时失效。
	
 		drop index idx_atb on book ; 
		alter table book add index idx_authroid (authorid) ;
		alter table book add index idx_typeid (typeid) ;
		explain select * from book where authorid*2 = 1 and typeid = 2 ;
```

##### （3）复合索引不能使用不等于（!=  <>）或is null (is not null)，否则自身以及右侧所有全部失效。复合索引中如果有>，则自身和右侧索引全部失效。

```
explain select * from book where authorid = 1 and typeid =2 ;

	-- SQL优化，是一种概率层面的优化。至于是否实际使用了我们的优化，需要通过explain进行推测。

	explain select * from book where authorid != 1 and typeid =2 ;
	explain select * from book where authorid != 1 and typeid !=2 ;
		
	
	体验概率情况(< > =)：原因是服务层中有SQL优化器，可能会影响我们的优化。
	drop index idx_typeid on book;
	drop index idx_authroid on book;
	alter table book add index idx_book_at (authorid,typeid);
	explain select * from book where authorid = 1 and typeid =2 ;--复合索引at全部使用
	explain select * from book where authorid > 1 and typeid =2 ; --复合索引中如果有>，则自身和右侧索引全部失效。
	explain select * from book where authorid = 1 and typeid >2 ;--复合索引at全部使用
	----明显的概率问题---
	explain select * from book where authorid < 1 and typeid =2 ;--复合索引at只用到了1个索引
	explain select * from book where authorid < 4 and typeid =2 ;--复合索引全部失效

	--我们学习索引优化 ，是一个大部分情况适用的结论，但由于SQL优化器等原因  该结论不是100%正确。
	--一般而言， 范围查询（> <  in），之后的索引失效。

```

##### （4）补救。尽量使用索引覆盖（using index）

```
（a,b,c）
	select a,b,c from xx..where a=  .. and b =.. ;

```

##### (5) like尽量以“常量”开头，不要以'%'开头，否则索引失效

```
select * from xx where name like '%x%' ; --name索引失效
	
	explain select * from teacher  where tname like '%x%'; --tname索引失效

	explain select * from teacher  where tname like 'x%';
 
	explain select tname from teacher  where tname like '%x%'; --如果必须使用like '%x%'进行模糊查询，可以使用索引覆盖 挽救一部分。

```

##### （6）尽量不要使用类型转换（显示、隐式），否则索引失效

```
explain select * from teacher where tname = 'abc' ;
	explain select * from teacher where tname = 123 ;//程序底层将 123 -> '123'，即进行了类型转换，因此索引失效
```

（7）尽量不要使用or，否则索引失效

```
explain select * from teacher where tname ='' or tcid >1 ; --将or左侧的tname 失效。

8.一些其他的优化方法
	（1）
	exist和in
	select ..from table where exist (子查询) ;
	select ..from table where 字段 in  (子查询) ;

	如果主查询的数据集大，则使用In   ,效率高。
	如果子查询的数据集大，则使用exist,效率高。	

	exist语法： 将主查询的结果，放到子查需结果中进行条件校验（看子查询是否有数据，如果有数据 则校验成功）  ，
		    如果 复合校验，则保留数据；

	select tname from teacher where exists (select * from teacher) ; 
	--等价于select tname from teacher

	
	select tname from teacher where exists (select * from teacher where tid =9999) ;
	
	in:
	select ..from table where tid in  (1,3,5) ;
	

	（2）order by 优化
	using filesort 有两种算法：双路排序、单路排序 （根据IO的次数）
	MySQL4.1之前 默认使用 双路排序；双路：扫描2次磁盘（1：从磁盘读取排序字段 ,对排序字段进行排序（在buffer中进行的排序）   2：扫描其他字段 ）
		--IO较消耗性能
	MySQL4.1之后 默认使用 单路排序  ： 只读取一次（全部字段），在buffer中进行排序。但种单路排序 会有一定的隐患 （不一定真的是“单路|1次IO”，有可能多次IO）。原因：如果数据量特别大，则无法 将所有字段的数据 一次性读取完毕，因此 会进行“分片读取、多次读取”。
		注意：单路排序 比双路排序 会占用更多的buffer。
			单路排序在使用时，如果数据大，可以考虑调大buffer的容量大小：  set max_length_for_sort_data = 1024  单位byte

	如果max_length_for_sort_data值太低，则mysql会自动从 单路->双路   （太低：需要排序的列的总大小超过了max_length_for_sort_data定义的字节数）

	提高order by查询的策略：
	a.选择使用单路、双路 ；调整buffer的容量大小；
	b.避免select * ...  
	c.复合索引 不要跨列使用 ，避免using filesort
	d.保证全部的排序字段 排序的一致性（都是升序 或 降序）
```

