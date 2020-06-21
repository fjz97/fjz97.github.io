---
title: tomcat问题的汇总
date: 2017-02-05 22:14:19
tags:
- tomcat
categories:
- 配置
---
在server.xml的`<Host>`标签中加入

```
<Context path="/site" docBase="c:\site" reloadable="true"> 
</Context>
```

其中path为引用资源时的路径，docBase为资源绝对路径。注意：路径后面不能多加一个`'/'`，否则访问不到。

---

在tomcat8.5.8版本下，`response.sendRedirect("")`不跳转，`response.sendRedirect("/")`却能跳转，换成tomcat8.0.0就解决了

---

war不能在tomcat运行时删除，否则会删除自动解压的工程。 你可以停止tomcat后删除war。

另：当你重新部署的时候，如果有与war文件相同的文件夹，就不会重新部署。