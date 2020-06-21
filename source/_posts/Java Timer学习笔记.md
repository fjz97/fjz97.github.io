---
title: Java Timer学习笔记
date: 2017-03-24 20:42:13
tags:
- java
categories:
- 技术
---
Timer是java1.3引入util包的一个类，作用是定时完成任务，每次开启一个线程进行任务，所以可以执行耗时操作，当然，如果要进行ui修改，请用Handler在ui线程中进行。

打开Timer类的源文件

![Timer](/images/18/Timer.png)

先不管TimerThread和TaskQueue两个辅助类，只看Timer类，简洁明了，除了构造方法外只剩下schedule和scheduleAtFixedRate两个方法，这也是Timer类的两个核心方法。

在讲这两个方法之前先解释一下TimerTask类，这是一个实现了Runnable的抽象类，这个抽象类加入了对run方法执行状态的判断。

1.schedule(TimerTask, long):在指定时间后执行TimerTask。

2.schedule(TimerTask, long, long):在指定时间后执行TimerTask，并以后一个指定时间为周期重复执行。

scheduleAtFixedRate与schedule不同之处在于前者是以开始时间为基准计算下一次执行时间，而后者以前一次执行的时间为基准计算下一次执行时间。什么意思呢，加入我们设定周期为5秒，开始时间为0秒后，而第一次执行却在第8秒，那么scheduleAtFixedRate方法会尽量在第10秒执行下一次，而schedule方法则会尽量在第13秒执行下一次。

所以说在需要保证20秒执行4次的情况下，使用scheduleAtFixedRate方法更有保障。