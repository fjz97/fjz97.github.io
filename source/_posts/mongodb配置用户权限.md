---
title: mongodb配置用户权限
date: 2017-11-26 17:05:07
tags:
- mongodb
categories:
- 配置
---
开启mongod服务后，进入mongo shell

```
mongo --port=###
```

切换到myblog数据库

```
> use myblog
switched to db myblog
>
```

创建用户

```
db.createUser(
{
	user: '###',
	pwd: '###',
    roles:
    [
      {
        role: 'dbOwner',
        db: 'myblog'
      }
    ]
  }
)
```

mongoose连接

```
mongodb: 'mongodb://name:pwd@ip:port/myblog'
```

添加用户权限

```
db.grantRolesToUser("<username>", [{ role: "<role-name>", db: "<db-name>" }]) 
```

撤销用户权限

```
db.revokeRolesFromUser("<username>", [{ role: "<role-name>", db: "<db-name>"}])  
```

编辑批处理命令

```
start mongod -f mongodb.conf
```

运行自定义配置项

```
dbpath=C:\mongodb\data\db
logpath=C:\mongodb\log\mongodb.log
bind_ip=127.0.0.1
port=###
auth=true
```

总结一下：auth=true即可为mongodb设定权限访问，每个数据库有自己的用户，我们需要做的就是在该数据库下创建用户并为用户配置权限。