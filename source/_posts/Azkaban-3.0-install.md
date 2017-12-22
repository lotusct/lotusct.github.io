---
title: Azkaban3.0安装教程
date: 2017-05-07 14:16:00
categories: 技术
mathjax: true
toc: true
comments: true
tags: Azkaban 大数据
#layout: timeline
---
本次安装azkaban的版本3.20.0-5,相对版本2.5有部分变化，此次安装two-server模式。<!--more-->
In version 3.0 we provide three modes: the stand alone "solo-server" mode, the heavier weight two server mode and distributed multiple-executor mode. The following describes the differences between the two modes.

三种模式

* solo-server模式：exec进程和web进程为同一个进程，存放元数据的数据库为H2；
* two-server模式：与之前的单机版本类似，exec进程和web进程分开，存放元数据的数据库为mysql；
* multiple-executor模式：exec进程和web进程在不同的机器上，存放元数据的数据库为mysql。

## 安装准备
1. [Azkaban官网](https://azkaban.github.io)
2. [软件下载地址](https://github.com/azkaban/azkaban)
3. [官方插件地址](https://github.com/azkaban/azkaban-plugins)
4. [官方文档地址](http://azkaban.github.io/azkaban/docs/latest)

## 安装过程
* 下载。git clone https://github.com/azkaban/azkaban.git 得到文件夹 azkaban；
* 编译。执行命令cd azkaban，在该目录下执行 ./gradlew installDist 生成一系列文件
* 拷贝。另新建目录mkdir azkaban-3.20.0，执行

    ```bash
    cp azkaban/azkaban-exec-server/build/distributions/azkaban-exec-server-3.20.0-5-g28fc94e7.tar.gz azkaban-3.20.0
    cp azkaban/azkaban-web-server/build/distributions/azkaban-web-server-3.20.0-5-g28fc94e7.tar.gz azkaban-3.20.0
    cp azkaban/azkaban-sql/build/distributions/azkaban-sql-3.20.0-5-g28fc94e7.tar.gz azkaban-3.20.0
    cd azkaban-3.20.0
    tar -zvxf azkaban-exec-server-3.20.0-5-g28fc94e7.tar.gz
    tar -zvxf azkaban-web-server-3.20.0-5-g28fc94e7.tar.gz
    tar -zvxf azkaban-sql-3.20.0-5-g28fc94e7.tar.gz
    ```
* 创建azkaban元数据库。在Mysql数据库中创建元数据。同时从外部导入mysql jar包到azkaban-exec-server-3.0.0/extlib/和azkaban-web-server-3.0.0/extlib/

    ```
    1) 以root用户登录mysql
    2) CREATE DATABASE azkaban;
    3) CREATE USER 'azkaban'@'%' IDENTIFIED BY 'azkaban';
    4) GRANT SELECT,INSERT,UPDATE,DELETE ON azkaban.* to 'azkaban'@'%' WITH GRANT OPTION;
    5) flush privileges;
    6) use azkaban;
    7) source /home/hadoop/azkaban/azkaban-sql-3.20.0-5-g28fc94e7/create-all-sql-3.0.0.sql
    ```
* 配置azkaban-web-server。执行mv azkaban-exec-server-3.20.0-5-g28fc94e7 azkaban-exec-server-3.20.0-5,且在该目录下执行keytool -keystore keystore -alias jetty -genkey -keyalg RSA生成keystore文件，配置SSL。

    ```
    1) cd azkaban-exec-server-3.20.0-5，此目录如下bin conf extlib lib plugins projects temp目录(如该目录下没有相应的文件夹，则直接在azkaban-solo-server目录下拷贝就行)。
    2) conf目录下存放azkaban.properties global.properties log4j.properties三个文件
    3) extlib目录下存放mysql-connector-java-5.1.41-bin.jar连接mysql的Java驱动包
    4) plugins目录下存放jobtypes/commonprivate.properties文件
    ```
    **azkaban.properties**

    ```
    #Azkaban
    default.timezone.id=America/Los_Angeles
    #Azkaban JobTypes Plugins
    azkaban.jobtype.plugin.dir=plugins/jobtypes
    #Loader for projects    
    executor.global.properties=conf/global.properties
    azkaban.project.dir=projects
    #Azkaban元数据库信息    
    database.type=mysql    
    mysql.port=3306    
    mysql.host=localhost
    mysql.database=azkaban    
    mysql.user=azkaban
    mysql.password=azkaban
    mysql.numconnections=100
    #Azkaban Executor settings
    executor.maxThreads=50
    executor.port=12321
    executor.flow.threads=30
    #JMX stats    
    jetty.connector.stats=true
    executor.connector.stats=true
    #uncomment to enable inmemory stats for azkaban
    #executor.metric.reports=true
    #executor.metric.milisecinterval.default=60000
    ```    
* 配置azkaban-exec-server.配置对应conf/azkaban.properties conf/global.properties conf/log4j.properties

    **log4j.properties**

    ```    
    log4j.rootLogger=INFO,C    
    log4j.appender.C=org.apache.log4j.ConsoleAppender
    log4j.appender.C.Target=System.err
    log4j.appender.C.layout=org.apache.log4j.PatternLayout
    log4j.appender.C.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
    ```    
* 启动。启动web-server和exec-server，执行bin/azkaban-executor-start.sh和bin/azkaban-web-start.sh命令。创建Project，两种方式：web界面创建和通过API创建（[方法](http://azkaban.github.io/azkaban/docs/latest/#ajax-api)）。
## 插件安装
* git clone https://github.com/azkaban/azkaban-plugins.git
* cd到对应目录下
* 执行ant


