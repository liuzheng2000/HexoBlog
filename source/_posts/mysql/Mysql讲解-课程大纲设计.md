---
title: Mysql讲解 课程大纲设计
abbrlink: 3330459815
date: 2021-11-29 10:32:00
tags:
  - Mysql
  - 数据库
categories: 
  - 数据库
  - Mysql 
---
# MYSQL

## 建立数据库

### 建库语句

- create database  IF NOT EXISTS TEST_DB DEFAULT CHARACTER Set UTF8 DEFAULT COLLATE utf8_bin;
- 创建一个测试数据库，命名为 test_db，指定其默认字符集为 utf8，默认校对规则为 utf8_bin

### 建表语句

- CREATE TABLE IF NOT EXISTS `Student`(
  `ID` INT UNSIGNED AUTO_INCREMENT,  -- 序号
  `Stu_ID` VARCHAR(100) NOT NULL ,   -- 学号
  `Stu_NAME` VARCHAR(100)  ,   -- 姓名
  `Stu_CourseID` VARCHAR(100) ,  -- 课程号
  `Stu_Grade` VARCHAR(40) ,  -- 成绩
  PRIMARY KEY ( `ID` )
  )ENGINE=InnoDB DEFAULT CHARSET=utf8;


	- 建立学生表

- CREATE TABLE IF NOT EXISTS `Teacher`(
  `ID` INT UNSIGNED AUTO_INCREMENT,  -- 序号
  `Tea_ID` VARCHAR(100) NOT NULL ,   -- 教师学号
  `Tea_NAME` VARCHAR(100)  ,   -- 教师姓名
  `Tea_CourseID` VARCHAR(100),  -- 教授课程编号
  PRIMARY KEY ( `ID` )
  )ENGINE=InnoDB DEFAULT CHARSET=utf8;

  - 建立教师表

- CREATE TABLE IF NOT EXISTS `Course`(
  `Cour_ID` VARCHAR(100) NOT NULL ,   -- 科目编号
  `Cour_NAME` VARCHAR(100) ,   -- 课程科目
  PRIMARY KEY ( `Cour_ID` )
  )ENGINE=InnoDB DEFAULT CHARSET=utf8;

  - 建立科目表

### 插入数据

- 逐条插入

  - INSERT INTO Course ( Cour_ID, Cour_NAME)
        VALUES
        ( 036611, '物理' );
        

INSERT INTO Course ( Cour_ID, Cour_NAME)
                       VALUES
                       ( 036612, '生物' );
                       
INSERT INTO Course ( Cour_ID, Cour_NAME)
                       VALUES
                       ( 036613, '化学' );

		- 课程表

- 批量插入

  - INSERT INTO Student ( Stu_ID, Stu_NAME,Stu_CourseID,Stu_Grade )
        VALUES
      ( 181114930, '小青',036611,90),
      ( 181114930, '小青',036612,50),
      ( 181114931, '小红',036612,82),
      ( 181114932, '小蓝',036613,85); 
  - INSERT INTO Teacher ( Tea_ID, Tea_NAME,Tea_CourseID )
        VALUES
        ( 159850, '王倩',036611),
        ( 159850, '王倩',036612),
        ( 159850, '王倩',036613),
        ( 159851, '王宇',036611),
        ( 159852, '王宇',036612),
        ( 159853, '王刚',036613);

- ETL工具

## 数据查询

### 简单查询

- 例如：查询学号为181114930的课程名称

  - select a.Stu_NAME,b.Cour_Name
    from student a left join course b on a.Stu_CourseID = b.Cour_ID
    where Stu_ID = '181114930';

- 例如：查询各个老师旗下学生的名字(老师教授课程与学生选报的课程)

  - select distinct a.Tea_Name ,c.Stu_Name 
    from Teacher a 
    left join course b on a.Tea_CourseID  = b.Cour_ID
    left join student c on b.Cour_ID =  c.Stu_CourseID 

### 函数查询

- 例如：查询各个科目的平均分且按照科目的平均分降序排列

  - select b.Cour_NAME , AVG(a.Stu_Grade) as Stu_Grade 
    from Student a 
    LEFT JOIN Course b on a.Stu_CourseID = b.Cour_ID
    group by Stu_CourseID,Cour_NAME
    ORDER BY Stu_Grade DESC

### 存储过程查询

- 例如：计算学生的平均分(传入值为学号)

  - delimiter $
    create procedure stu_AvgGrade(in StudentID varchar(100))
    begin
    select avg(Stu_Grade)  from Student where Stu_ID = StudentID;
    end $

delimiter ;

call stu_AvgGrade('181114930');

## 数据库导入

### 此处使用的是Excel导入，在企业中，应该是使用ETL工具导入

## 数据库备份与恢复

## 数据库日志

## ETL

## 帆软

*XMind - Trial Version*