---
title: Linux运行JAR
tags: Linux
categories: Linux
abbrlink: 4164729442
date: 2021-12-23 16:19:14
---
Linux 运行jar包命令如下：

方式一：

1. java -jar shareniu.jar

特点：当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出

那如何让窗口不锁定？

方式二

1. java -jar shareniu.jar &

&代表在后台运行。

特定：当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。

继续改进，如何让窗口关闭时，程序仍然运行？

方式三

nohup java -jar shareniu.jar &

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行

当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件。

方式四

1. nohup java -jar shareniu.jar >temp.txt &

解释下 >temp.txt

command >out.file

command >out.file是将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中。

可通过jobs命令查看后台运行任务

1. jobs

那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。
 如果想将某个作业调回前台控制，只需要 fg + 编号即可。

1. fg 23

查看某端口占用的线程的pid

netstat -nlp |grep :9181