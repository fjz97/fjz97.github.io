---
title: MySQL触发器（trigger）约束
date: 2017-05-19 15:00:41
tags:
- mysql
categories:
- 技术
---
今天数据库上机学习了sql server的五种约束：NOT NULL,UNIQUE,PRIMARY KEY,FOREIGN KEY,CHECK。其中前四个约束在MySQL中都有直接体现，NOT NULL和主键外键自不用说，UNIQUE对应了UNIQUE索引。然而MySQL却不支持CHECK约束，OMG，这点还是微软做得好啊~

解决之道何在？？在MySQL里，就要用trigger来取代CHECK约束了（sql server里也有）。就以数据库课设为例，我们有一张员工表，里面有字段“性别”，性别当然只有男女两个选项，考虑到在插入前就要对性别字段做检查，我们选择BEFORE，插入，定义如下（navicat环境）：

```
begin
if (NEW.性别 != '男' && NEW.性别 != '女')
then delete from employee;
end if;
end
```

是不是很像函数？？它的本质就是在指定条件下自动触发的函数（虽然我没用过函数233）。

现在，添加性别为“秀吉”的数据后，服务器出错返回错误了！