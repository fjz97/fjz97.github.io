---
title: jQuery学习笔记2：性能和技巧
date: 2017-02-02 18:06:09
tags:
- jQuery
categories:
- 技术
---
1、选择器的选用
-----

尽量使用`$("#id")`，会调用js底层的`document.getElementById()`方法，速度最快。

其次使用标签选择器`$("p")`，速度也很快。

接下来是`$(".class")`，再后是`$(":hidden")`。

后者性能很低，最好用`$("#content").find(":hidden")`或是`$("a.button").filter(":animated")`来代替。

-----

2、尽量缓存对象
-----

**原则：重复出现的元素用变量缓存下来，防止jQuery创建多个对象占用内存**。

-----

3、不要在循环内操作DOM或者其他耗时操作
-----

错误例子如下

```
for(var i = 0; i < 100; i++){
    $list = $("#list");
    $list.append("<li>" + i + "</li>");
}
```

上面代码段有两处地方欠妥：在循环中查找元素，在循环中操作DOM。

-----

4、使用事件代理
-----

每个js事件都会冒泡到父级节点，当给多个元素元素调用同一个函数时，可以对父对象进行事件绑定。

例如，给table中的td绑定点击背景变红事件时，可以这样写。

```
$("#table").click(function(e){
    var $clicked = $(e.target);
    $clicked.css("background", "red");
})
```

-----

5、使用$.proxy()指定上下文
-----

```
$("#panel").fadeIn(function(){
    $("#panel button").click($.proxy(function(){
        $(this).fadeOut();
    }, this));
})
```

click触发后panel消失。