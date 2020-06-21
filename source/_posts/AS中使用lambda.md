---
title: AS中使用lambda
date: 2017-02-20 13:01:00
tags:
- Android
categories:
- 配置
---
在项目gradle中加入

`classpath 'me.tatarka:gradle-retrolambda:3.2.5'`

在app模块gradle中加入

```
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

以及

`apply plugin: 'me.tatarka.retrolambda'`

引入插件