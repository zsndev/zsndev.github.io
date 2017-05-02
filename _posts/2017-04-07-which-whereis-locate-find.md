---
layout: post
title: which、whereis、find的区别
date: 2017-04-07
tags: linux    
---

### which  
语法：   
which 可执行文件名称  
例如：
[root@redhat ~]# which ls  
```
alias ls='ls --color=auto'
/bin/ls
```
在PATH变量里面寻找,一般查可执行文件和别名(alias)

### whereis
语法：   
```
whereis [-bmsu] 文件或者目录名称
参数说明：   
-b ： 只找二进制文件   
-m： 只找在说明文件manual路径下的文件  
-s ： 只找source源文件   
-u ： 没有说明文档的文件  
```
例如：   
[root@redhat ~]# whereis ls 
`ls: /bin/ls /usr/share/man/man1/ls.1.gz`  
从linux文件数据库（/var/lib/slocate/slocate.db 或 /var/lib/mlocate/mlocate.db）寻找，所以有可能找到刚刚删除，或者没有发现新建的文件

### find
语法：   
find 路径 参数  

参数说明：   
```
时间查找参数：   
-atime n :将n*24小时内存取过的的文件列出来   
-ctime n :将n*24小时内改变、新增的文件或者目录列出来    
-mtime n :将n*24小时内修改过的文件或者目录列出来   
-newer file ：把比file还要新的文件列出来   

名称查找参数：   
-gid n       ：寻找群组ID为n的文件   
-group name  ：寻找群组名称为name的文件   
-uid n       ：寻找拥有者ID为n的文件   
-user name   ：寻找用户者名称为name的文件   
-name file   ：寻找文件名为file的文件（可以使用通配符）   
```
例如：
`[root@redhat ~]# find / -name xxx`  

用whereis和locate无法查找到我们需要的文件时，可以使用find，但是find是在硬盘上遍历查找
