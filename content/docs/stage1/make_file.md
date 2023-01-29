---
title: "Make File"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

概述
makefile关系到了整个系统的编译规则，哪些文件需要先编译，哪些后编译等等
自动化编译
Makefile的规则

如果prerequisites后面的文件比target文件新，就执行command
执行：make
一层层地找文件的依赖关系，直到最终编译出第一个目标文件
变量
 $(obj)
自动推导
make看到.o文件会自动把.c文件加在依赖关系prerequisites中
clean
make clean 才会执行
放在文件的最后