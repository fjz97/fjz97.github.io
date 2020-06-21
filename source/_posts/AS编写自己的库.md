---
title: AS编写自己的库
date: 2017-04-15 19:39:50
tags:
- Android
categories:
- 配置
---
1.编写android library
---

除了我们熟知的jar包外，android还支持另外一种打包——aar包，它与jar包不同之处在于它可以把资源文件一起打包，而jar包只打包源代码。

1.点击file->new->new module->android library，发现我们的项目下多了一个文件，gradle脚本也多了一个文件。

![project](/images/23/project.png)

实质上我们的android library是以新的module的形式创建的，它与原本项目的app module是一样的，唯一不同之处在于它的module的gradle文件的第一行是

`apply plugin: 'com.android.library'`

这样以后在项目build后会在它的build文件夹下生成aar文件而不是apk文件。

打开project的gradle文件，include的module从原来的app变成了app和mylibrary，说明我们确实给项目增加了module。

`include ':app', ':mylibrary'`

如果我们想删除这个库，直接从project的gradle文件中删除它的module就可以了，剩下的移除工作全由AS解决。

2.编写完android库后，回到app module中却发现写的库并不可用，原因是app module并没有引入库module。

点击file->project structure->app->dependencies->add->3.module dependencies->mylibrary就可以把库module添加到app module的依赖中去了。

app module的gradle文件中多了这么一行

`compile project(":mylibrary")`

3.如果在别的项目中想要使用库，有两种办法

①引入module。好处在于可以更改库中的文件，更加灵活。点击file->new->import module选择要引入的module，重复2的操作在app module中引入依赖。

②引入aar包。我们将库所在文件夹的build文件里找到aar包，复制到要使用库的项目的libs文件夹下，按照先前的经验，接下来应该在app module的gradle文件中增加aar依赖。

`compile(name:'mylibrary', ext:'aar')`

之后的思路也很明确，AS如何能找到mylibrary文件呢，libs只是一个普通文件夹而已，并没有环境变量定义它的路径，办法是给app module增加一个仓库，仓库地址就是libs文件夹。

```
repositories {
    flatDir {
        dirs 'libs'
    }
}
```

OK，可以愉快地在项目中使用我们自己编写的android库啦~

2.创建自己的gradle依赖包
---

参考：[Android创建自己的gradle依赖包](https://blog.csdn.net/tgbus18990140382/article/details/53066865)

首先这里需要用到两个网站：

1.github

2.jitpack

具体步骤

1.在AS创建好我们的android library后，使用AS自带的VCS插件关联到github仓库（自己创建仓库也没关系）。

2.在github中发布release版本，进入jitpack，生成依赖包。

3.在项目中加入gradle依赖。