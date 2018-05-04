---
title: Maven
toc: true
date: 2018-03-29 23:30:44
categories:
- tool
tags: [Tool, Java, Maven]
---

依赖范围与Classpath的关系
|scope|对编译classpath有效|对测试classpath有效|对运行时classpath有效|例子|
| :-: | :-: | :-: | :-: | :-: |
|compile|Y|Y|Y|spring-core|
|test|-|Y|N|JUnit|
|provided|Y|Y|-|javax-servlet-api|
|runtime|-|Y|Y|mysql-connector-java（MySQL的jdbc驱动）|
|system|Y|Y|-|本地地、Maven仓库之外的类库文件|

注：
- compile为默认的scope
- import (Maven2.0.9及以上)的依赖范围不会对三者classpath造成实际影响
