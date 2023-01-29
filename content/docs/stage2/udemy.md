---
title: "Udemy"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

Section 1: 持续集成和Jenkins介绍
1. 持续集成

在开发过程中尽早地发现bug
持续集成、持续交付、持续部署的区别

持续集成：个人研发的部分向软件整体的部分交付。频繁进行集成可以更快地发现其中的问题
持续交付：将集成后的代码交付到类生产环境中
持续部署：自动部署到生产环境中
2. Jenkins介绍
Jenkins是持续集成和构建服务器
Java写的，开源的
Section 2: 环境配置和Jenkins安装
java version：1.8.0 （Java 8）

Jenkins version: 2.121.3，这个版本很多plugin用不了了，所以改用2.346.1



Section 3: Jenkins架构、术语、UI介绍、创建第一个Jenkins Job
Master和Slave架构


master改成main了

关键术语
Job/Project: 由Jenkins监视或控制的任务
Slave/Node: node是所有的机器，包括slave和master
slave agent：jenkins在slave上运行一个单独的程序
slave注册到master之后，master开始向slave分配负载
Executor: 独立的构建流，一个node可以有>=1个executor
Build: 一个build是一个job/project的结果
Plugin: 拓展核心jenkins服务核心功能的软件
Jenkins UI
manage plugin报错怎么办

创建第一个Jenkins Job
运行第一个Jenkins Job
注意manage Jenkins -> manage plugin →advanced 中的HTTP Proxy Configuration

enter image description here

Section 4: 用Jenkins持续集成
安装Jenkins GitHub plugin
第一个基于maven的Jenkins项目
Maven
什么是maven
描述软件是如何构建的
描述项目的依赖
安装的时候注意设置PATH
Maven中的pom.xml文件：描述正在构建的软件项目
Maven构建软件生命周期中的不同阶段
validate, compile, test, package, verify, install, deploy
只需要调用要执行的最后一个构建阶段
注意要给maven设置proxy

要把里面的username和userpasswd删掉
在Jenkins中进行源代码控制轮询
Cron语法


SCM means source code management

几个触发器
build periodically：不集成，只是执行任务
Section 5: Jenkins 持续检查
代码质量和代码覆盖率度量报告
check style: java代码静态分析工具
已经下架了，现在改用 warning next generation了
configuration里这样写

还有一些其他的静态分析工具
PMD
FindBugs
Jenkins支持的其他构建系统
Ant, Gradle, Shell脚本

Section 6: Jenkins 持续交付
归档构建artifacts
staging环境上安装和配置tomcat
部署到staging环境上

线性管道
Jenkins构建pipeline
可视化界面

并行的进行Jenkins构建
->
checkstyle可能花费很长时间
让staging尽快开始

部署到生产环境上

必须手动触发
如何在一个机器上同时运行两个tomcat程序：运行在不同的端口
staging: port 8091
production: port 9091

Section 7: Jenkins Pipeline as Code (Jenkinsfile)
把Web应用程序管道转化成代码
版本控制
执行job不容易漏掉步骤
Jenkinsfile：领域专用语言
代码格式介绍

创建Pipeline Job
Jenkinsfile handbook
parallel 并行运行任务
Section 8: Jenkins和Docker集成
先跳过了

Section 9: 分布式构建


需要master节点能够SSH免密登录到slave节点
以我的VM作为master节点，NUC作为slave节点
Jenkins上新建节点
给节点加上标签，使得job运行在特定标签的节点（机器）上
在node configure中