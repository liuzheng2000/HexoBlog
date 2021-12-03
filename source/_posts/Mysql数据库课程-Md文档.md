---
title: Mysql数据库课程 Md文档
abbrlink: 2655269665
date: 2021-11-29 10:39:20
tags:
  - Mysql
  - 数据库
categories: 
  - 数据库
  - Mysql 
---
## 数据库
```sql
CREATE TABLE IF NOT EXISTS `Student`(
   `ID` INT UNSIGNED AUTO_INCREMENT,  -- 序号
   `Stu_ID` VARCHAR(100) NOT NULL ,   -- 学号
   `Stu_NAME` VARCHAR(100)  ,   -- 姓名
   `Stu_CourseID` VARCHAR(100) ,  -- 课程号
   `Stu_Grade` VARCHAR(40) ,  -- 成绩
   PRIMARY KEY ( `ID` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

```



```sql
INSERT INTO Student ( Stu_ID, Stu_NAME,Stu_CourseID,Stu_Grade )
                      VALUES
                    ( 181114930, '小青',036611,90),
                    ( 181114930, '小青',036612,50),
                    ( 181114931, '小红',036612,82),
                    ( 181114932, '小蓝',036613,85); 
```



```sql
CREATE TABLE IF NOT EXISTS `Teacher`(
   `ID` INT UNSIGNED AUTO_INCREMENT,  -- 序号
   `Tea_ID` VARCHAR(100) NOT NULL ,   -- 教师学号
   `Tea_NAME` VARCHAR(100)  ,   -- 教师姓名
   `Tea_CourseID` VARCHAR(100),  -- 教授课程编号
   PRIMARY KEY ( `ID` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



```sql
INSERT INTO Teacher ( Tea_ID, Tea_NAME,Tea_CourseID )
                       VALUES
                       ( 159850, '王倩',036611),
                       ( 159850, '王倩',036612),
                       ( 159850, '王倩',036613),
                       ( 159851, '王宇',036611),
                       ( 159852, '王宇',036612),
                       ( 159853, '王刚',036613);
```



```sql
CREATE TABLE IF NOT EXISTS `Course`(
   `Cour_ID` VARCHAR(100) NOT NULL ,   -- 课程编号
   `Cour_NAME` VARCHAR(100) ,   -- 课程科目
   PRIMARY KEY ( `Cour_ID` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



```sql
INSERT INTO Course ( Cour_ID, Cour_NAME)
                       VALUES
                       ( 036611, '物理' );
                       
INSERT INTO Course ( Cour_ID, Cour_NAME)
                       VALUES
                       ( 036612, '生物' );
                       
INSERT INTO Course ( Cour_ID, Cour_NAME)
                       VALUES
                       ( 036613, '化学' );
```
## 数据库导入
- 数据库导入
  - ![image-20210902191100639](http://typa.qingyun.run/img/image-20210902191100639.png)

- 选择导入向导
- 选择Excel文件
- 选择文件位置
- 后续基本就是直接导入
- 如图
- ![image-20210902191403193](http://typa.qingyun.run/img/image-20210902191403193.png)

![image-20210902191532235](http://typa.qingyun.run/img/image-20210902191532235.png)

头名不建议使用中文，后续手动修改

## 数据库查询
例如：查询学号为181114930的课程名称

```sql
select a.Stu_NAME,b.Cour_Name
from student a left join course b on a.Stu_CourseID = b.Cour_ID
where Stu_ID = '181114930';
```



例如：查询各个老师旗下学生的名字(老师教授课程与学生选报的课程)

``` sql
select distinct a.Tea_Name ,c.Stu_Name 
from Teacher a 
left join course b on a.Tea_CourseID  = b.Cour_ID
left join student c on b.Cour_ID =  c.Stu_CourseID 
```



例如：查询各个科目的平均分且按照科目的平均分降序排列

```sql
select b.Cour_NAME , AVG(a.Stu_Grade) as Stu_Grade 
from Student a 
LEFT JOIN Course b on a.Stu_CourseID = b.Cour_ID
group by Stu_CourseID,Cour_NAME
ORDER BY Stu_Grade DESC
```



例如：计算学生的平均分(传入值为学号)

```sql
delimiter $
create procedure stu_AvgGrade(in StudentID varchar(100))
begin
    select avg(Stu_Grade)  from Student where Stu_ID = StudentID;
end $

delimiter ;

call stu_AvgGrade('181114930');
```
## 简历触发器
创建测试表

```sql
CREATE TABLE IF NOT EXISTS `triggertest`(
   `ID` INT ,  -- 序号
    `Name` varchar(100), -- 名字
   `createDate` datetime ,   -- 创建时间
  `updateDate` datetime    -- 修改时间
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

查询所有触发器（当前库）

```sql
show TRIGGERS
```

建立触发器（插入数据时，自动生成插入的时间）

```sql
DELIMITER $$
 
CREATE
    TRIGGER `create_time` BEFORE INSERT
    ON `triggertest`
    FOR EACH ROW BEGIN
	SET new.createDate=NOW();
    END$$
 
DELIMITER ;
```



修改数据时，自动生成修改的时间

```sql
DELIMITER $$
 
CREATE
    TRIGGER `update_time` BEFORE UPDATE
    ON `triggertest`
    FOR EACH ROW BEGIN
	SET new.updateDate=NOW();
    END$$
 
DELIMITER ;
```

## 数据库定时备份

- 优先执行一次全量备份(此处为导出数据库的所有数据结构与数据内容)

- ```bash
  D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysqldump --flush-logs --single-transaction --master-data=2 -u root -p test_db  > D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\log\backup.sql
  ```

  ```
      参数 --lock-all-tables
  
  对于InnoDB将替换为 --single-transaction。
  该选项在导出数据之前提交一个 BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于事务表，例如 InnoDB 和 BDB。本选项和 --lock-tables 选项是互斥的，因为 LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用 --quick 选项。
  
      参数 --flush-logs，结束当前日志，生成并使用新日志文件
  
      参数 --master-data=2，该选项将会在输出SQL中记录下完全备份后新日志文件的名称，用于日后恢复时参考，例如输出的备份SQL文件中含有：CHANGE MASTER TO MASTER_LOG_FILE='MySQL-bin.000002', MASTER_LOG_POS=106;
  
      参数 test，该处的test表示数据库test，如果想要将所有的数据库备份，可以换成参数 --all-databases
  
      参数 --databases 指定多个数据库
  
      参数 --quick或-q，该选项在导出大表时很有用，它强制 MySQLdump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。
  
      参数 --ignore-table，忽略某个数据表，如 --ignore-table test.user 忽略数据库test里的user表
  
      更多mysqldump 参数，请参考网址
  
  ```

- 全量数据库恢复

- ```sql
  D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysql -u  root  -proot test_db   <  D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\log\backup.sql
  ```

查询当前数据库日志版本

```sql
show master status;
```

刷新增量日志版本

```bash
D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysqladmin -uroot -proot flush-logs
```

恢复日志

```bash
D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysqlbinlog --no-defaults D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\data\binlog.000003 | D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysql -uroot -proot test_db
```

## 数据库索引
- 通过存储过程批量插入数据

  ```sql
  delimiter $
  create procedure InstertData(in Num int(10))
  begin
  declare i int;
  set i=1;
  while  i < Num  do
  insert into initdata(Sno) values(2);
  set  i=i+1;
  end while ;
  end $
  delimiter ;
  ```

- 数据库索引

  可以采用导入的数据

  执行语句

  ```sql
  select a.major,a.instructor,a.SName from initdata a where major = "会计学" and instructor = "靳祺" 
  ```

  ![image-20210902211120473](http://typa.qingyun.run/img/image-20210902211120473.png)

不建立索引的情况下  执行时间为0.44s

建立聚合索引

```sql
create index selectName On indata(major,instructor,SName)
```

![image-20210902211314332](http://typa.qingyun.run/img/image-20210902211314332.png)

符合索引条件

执行时间为0.25s

近一倍的时间差值



索引在工作中非常重要，但是难以学习，需要花费很深的功夫去研究！（mysql 优化机制 ）

## 增量备份
### 增量备份

#### 1. 检查log_bin是否开启

进入mysql命令行，执行 `show variables like '%log_bin%' `

```
mysql> show variables like '%log_bin%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin                         | OFF   |
| log_bin_basename                |       |
| log_bin_index                   |       |
| log_bin_trust_function_creators | OFF   |
| log_bin_use_v1_row_events       | OFF   |
| sql_log_bin                     | ON    |
+---------------------------------+-------+
6 rows in set (0.01 sec)
```

如上所示，log_bin 未开启；如果log_bin开启，则跳过第2步，直接进入第3步。

#### 2. 开启 log_bin，并重启mysql

- 编辑 mysql 的配置文件 `vim /etc/my.cnf`，在 mysqld 下面添加下面2条配置

```
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
server_id=152
```

Tip1: 一定要加 server_id，否则会报错。至于server_id的值，随便设就可以。
 Tip2: log_bin 中间可以下划线_相连，也可以-减号相连。同理server_id也一样。

- 重启mysql

```
service mysqld restart
```

- 再次在mysql命令行中执行 `show variables like '%log_bin%'`

```
mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/mysql-bin       |
| log_bin_index                   | /var/lib/mysql/mysql-bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
6 rows in set (0.01 sec)
```

## SQL
/*
 Navicat Premium Data Transfer

 Source Server         : mysql
 Source Server Type    : MySQL
 Source Server Version : 80020
 Source Host           : localhost:3306
 Source Schema         : test_db

 Target Server Type    : MySQL
 Target Server Version : 80020
 File Encoding         : 65001

 Date: 02/09/2021 19:31:31
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for data
-- ----------------------------
DROP TABLE IF EXISTS `data`;
CREATE TABLE `data`  (
  `ID` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `major` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `instructor` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `instructor_Tell` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `class` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `Sno` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `SName` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `SParentName` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL,
  `address` varchar(255) CHARACTER SET utf8 COLLATE utf8_bin NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_bin ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;

## Mysql全量备份
D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysqldump --flush-logs --single-transaction --master-data=3 -u root -proot test_db  > D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\log\backup.sql

## mysql增量备份
D:\mysql-8.0.18-winx64\mysql-8.0.20-winx64\mysql-8.0.20-winx64\bin\mysqladmin -uroot -proot flush-logs