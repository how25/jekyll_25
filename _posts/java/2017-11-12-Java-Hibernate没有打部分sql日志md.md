---
layout: post
categorise: java hibernate
tag: java hibernate
---
# Hibernate 项目执行过程中没有输出需要执行的 *sql* 日志

##### 因为没有执行 *sql* 导致 *dao* 返回了数据为空 引起下面代码报空指针异常

##### 主要原因是 *hibernate* 没有扫到对应的实体类包

##### 但是该问题不会造成程序报错，只是不会执行相关 *sql* 返回的数据为空

##### 只要在对应的配置文件中加入对应实体类的扫包路径即可解决







