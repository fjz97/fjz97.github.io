---
title: java File返回什么路径
date: 2018-05-16 21:15:45
tags:
- java
categories:
- 技术
---
起因
===

Android7.0为了进一步加强私有文件的安全性，不再由开发者放宽私有文件的访问权限，这就意味着我们在代码中不能再随心所欲地操纵一些文件。

昨天在做更新app并打开安装包的时候，就报了FileUriExposedException 异常。经过百度，找到了解决的办法——使用FileProvider，[参考文章在这里](https://www.jianshu.com/p/3f9e3fc38eae)。照着文章使用了FileProvider后，却出现了解析程序包时出现问题。

![解析程序包时出现问题](/images/26/problem.png)

查看log发现了如下的warning：

```
Error staging apk from content URI: content://com.fengjingzhe.fightpicture.fileprovider/download/file%3A/storage/emulated/0/Android/data/com.fengjingzhe.fightpicture/files/Download/fightpicture-1.1.2.apk
                                                     java.io.FileNotFoundException: No such file or directory
```

也就是说，uri出了错误，系统找不到apk文件了。我赶紧查看生成uri的代码。

```
Uri apkUri = FileProvider.getUriForFile(MainActivity.this, "com.fengjingzhe.fightpicture.fileprovider", file);
```

这个uri是通过官方的api生成的，网上也都是使用了这个方法，理应不会有错猜对啊！！！真是百思不得其解！！！

反复观察了这个生成的uri，我猜测问题出在了中间的`file%3A`上。抱着试一试的心态，我自己拼接了一下uri：

`content://com.fengjingzhe.fightpicture.fileprovider/download/storage/emulated/0/Android/data/com.fengjingzhe.fightpicture/files/Download/fightpicture-1.1.2.apk`

运行程序，居然成功了！成功打开了这个apk，进入安装界面。

那么为什么官方的api生成的uri出错了呢？我深入FileProvider源码仔细查看，看了大半个晚上，仍然没有找出猫腻，只能悻悻然会宿舍了。

解决
===

时间回到今天，我在测试`File.getAbsolutePath()`和`File.getPath()`两个方法返回的路径究竟有什么区别的时候，为了省事，直接在项目中找现成的File类进行测试，其中就包括了这个下载好的apk。这时，意料之外的情况发生了。我发现，不同的两个文件，就算是同一个`getPath()`方法，居然返回的路径格式大相径庭。比如，这个apk文件返回了这样一个路径：

`file:/storage/emulated/0/Android/data/com.fengjingzhe.fightpicture/files/Download/fightpicture-1.1.2.apk`

而一张图片文件返回了这样一个路径：

`/storage/emulated/0/fightpicture/1526470886506.jpg`

抱着疑问，我打开了File的源码，这里贴出关键部分：

```
    public File(String pathname) {
        if (pathname == null) {
            throw new NullPointerException();
        }
        //nomalize方法去除了头尾多余的斜杠
        this.path = fs.normalize(pathname);
        this.prefixLength = fs.prefixLength(this.path);
    }

	public String getPath() {
        return path;
    }
```

也就是说，你的构造方法传什么样的路径，getPath()方法就会返回给你什么样的路径，而构造方法恰恰接受file:///和/开头的两种类型路径。

回到getUriForFile方法上来，之所以返回的uri多了一点东西，就是因为我们的apk文件的getPath方法返回的路径多了file:///的前缀。那么，我们在getUriForFile方法中给它传一个/前缀的文件，返回的uri是不是就正常了呢？我们改动代码：

```
File file = new File(Uri.parse(filename).getEncodedPath());
                    Uri apkUri = FileProvider.getUriForFile(MainActivity.this, "com.fengjingzhe.fightpicture.fileprovider", file);
```

果然，成功打开了apk文件，进入了安装界面。问题成功解决了。