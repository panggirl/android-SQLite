# Sqlite 语句收集
sqlite支持建立自增主键，sql语句如下： 
CREATE TABLE w_user( 
id integer primary key autoincrement, 
userename varchar(32), 
); 

2、联合主键 
在建表时 
CREATE TABLE tb_test ( 
bh varchar(5), 
id integer, 
ch varchar(20),
mm varchar(20),
primary key (id,bh)); 
注意：在创建联合主键时，主键创建要放在所有字段最后面，否则也会创建失败

3、外键
create table Book(title TEXT,user_id INTEGER not Null,id INTEGER not Null PRIMARY KEY,
FOREIGN KEY (user_id ) REFERENCES User(id) on Delete CASCADE )

4、查询表是否存在
SELECT COUNT(*) FROM sqlite_master where type='table' and name='表名'

5、插入语句
insert into Book (title,user_id) values('xiao wang',1);
