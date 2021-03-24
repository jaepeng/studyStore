# Sqlite

## Sqlite简介

1. 无数据类型
2. 嵌入式的
3. 可嵌入式的
4. 支持事务操作——提高运行速度
5. 跨平台的磁盘文件
6. api简单易用

### Sqlite介绍

#### 数据库类型

1. Integer
2. varChar（10）
3. float
4. double
5. char(10)
6. text

#### sql回顾：

##### 创建表的语句

> create table 表名(字段名称 数据类型 约束（是否主键...)，字段名称 数据类型 约束....)
>
> create table person(_id Integer primary key,name varchar(10),age Integer not null)

##### 删除表的语句

> drop table  表名
>
> drop table person

#### 插入数据

> insert into 表明[字段，字段] values（值一，值二...)
>
> insert into person(_id,age) values(1,20)
>
> insert into pserson values(2,"zs",30)//从表结构的第一个开始依次插入

#### 修改数据

> update 表名 set 字段 =新值 where 修改的条件
>
> update person set name="ls" where _id=1
>
> 如果没有where语句，代表对所有数据进行修改
>
> 如果想要修改多个值：
>
> update person set name="ls"，age=20 where _id=1

#### 删除数据（清除表中的数据）

> delete from 表名 where 删除的条件
>
> delete from person where _id=2
>
> 同样，如果没有where语句，代表删除表中所有记录

#### 查询语句

>  select 字段名 from 表名 where 查询条件 group by 分组的字段 having 筛选条件 order by 排序字段

#### 左连接：

按照条件语句，左边表有的数据都得有，没有的属性就为null

## Android中的数据库

getReadableDatabase  getWritableDatabase

两者的区别：

1. 创建或打开数据库
2. 默认情况下，两者都表示打开或创建可读可写数据库
3. 如果磁盘已满，或是数据库本身权限不足等情况下getReadableDatabase打开的就是只读数据库

## Sqlite查询

### rawQuery()

sql语句

```java
public static Cursor selectDataBySql(SQLiteDatabase db, String sql, String[] selectionArgs){
    Cursor cursor=null;
    if (db!=null){
        cursor = db.rawQuery(sql, selectionArgs);
    }
    return cursor;
}
```

Api查询

```java
db = mHelper.getWritableDatabase();
//
cursor = db.query(Constans.TABLE_NAME, null, Constans._ID+">?", new String[]{"2"}, null, null, Constans._ID + " desc");
List<Person> list1 = DbManager.cursorToList(cursor);
for (Person person : list1) {
    Log.d(TAG, "onClick: "+person.toString());
}
db.close();
```

## Sqlite适配器

+ SimpleCursorAdapter 主键必须叫 _id 不然会报错.

+ CursorAdapter



## Sqlite分页

![image-20201211145957438](https://gitee.com/pengjae/pic/raw/master/img/20201211150004.png)





