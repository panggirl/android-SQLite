# 接手项目上使用的 SugarORM 
[https://github.com/chennaione/sugar](https://github.com/chennaione/sugar)
```
//映射数据库
    implementation 'com.github.satyan:sugar:1.5'
```

## 入坑
```
数据库的更新操作：

1.在assets目录中创建一个sugar_upgrades的文件夹

2.创建升级版本的文件，命名规范是.sql。例如 1.sql 2.sql

3.然后将你manifest里面里面的version改为对应的升级版本号

4.如果目前是1版本，现在要更新到4版本，suagr会从2.sql，3.sql，4.sql挨个去查询你的更新记录

5.最关心的来了，就是sql文件里面的内容是啥呢？？ 支持脚本语言！！！

例如，你某一次更新想在book表里加一个字段只需要这样

alter table BOOK add NAME TEXT;

6.最后说一句，upgrade的代码每一行都要以“；”结尾！！！

**原话在此： 
This is a normal sql script file. You can add all your alter and insert/update queries, 
one line at a time. Each line is terminated by a ; (semicolon).**
```
业务上需要添加字段升级数据库，按照上面的规则，添加多个字段，SQL语句:
```
ALTER TABLE mytable ADD (source INTEGER DEFAULT 0, time TEXT, dura TEXT);
或
ALTER TABLE mytable ADD source INTEGER DEFAULT 0, time TEXT, dura TEXT;
```
编译ok，然后运行有关数据库业务报错：
```
2019-04-13 13:02:17.296 30331-30331/com.fupai.testdata E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.fupai.testdata, PID: 30331
    android.database.sqlite.SQLiteException: duplicate column name: source (code 1): , while compiling: 
    ALTER TABLE mytable ADD (source INTEGER DEFAULT 0, time TEXT, dura TEXT)
        at android.database.sqlite.SQLiteConnection.nativePrepareStatement(Native Method)
        at android.database.sqlite.SQLiteConnection.acquirePreparedStatement(SQLiteConnection.java:965)
        at android.database.sqlite.SQLiteConnection.prepare(SQLiteConnection.java:576)
        at android.database.sqlite.SQLiteSession.prepare(SQLiteSession.java:588)
        at android.database.sqlite.SQLiteProgram.<init>(SQLiteProgram.java:58)
        at android.database.sqlite.SQLiteStatement.<init>(SQLiteStatement.java:31)
        at android.database.sqlite.SQLiteDatabase.executeSql(SQLiteDatabase.java:1713)
        at android.database.sqlite.SQLiteDatabase.execSQL(SQLiteDatabase.java:1643)
        at com.orm.SchemaGenerator.executeScript(SchemaGenerator.java:128)
        at com.orm.SchemaGenerator.executeSugarUpgrade(SchemaGenerator.java:100)
        at com.orm.SchemaGenerator.doUpgrade(SchemaGenerator.java:64)
        at com.orm.SugarDb.onUpgrade(SugarDb.java:33)
        at android.database.sqlite.SQLiteOpenHelper.getDatabas
```
后来改为添加一个字段也如此：
后来查阅 [官方文档-是 1.3版本](http://satyan.github.io/sugar/)也找不到问题原因，后来查看 
[issues-Duplicate column name error after automatic migration #713 ](https://github.com/chennaione/sugar/issues/713)，
发现如下：
```
Hey,
Sorry about the problem.
Interestingly though, I simply changed my entity classes, changed the db version value and didn't write any upgrade scripts.
It did however automatically add the columns, despite the docs saying you need to write the upgrade scripts.
So try doing away with the upgrade scripts.
I'm here if things don't work out. :)
```
大概就是他更改了数据库版本号和实体类但没有添加相应的 version.sql 脚本也实现数据库正常升级，证明 1.5 版本它确实可以自动添加列而不需要写SQL脚本，
然后我照做真的可以！坑啊

注：官方文档中嵌套实体类我没成功，实体类属性命名最好以驼峰规则写。
