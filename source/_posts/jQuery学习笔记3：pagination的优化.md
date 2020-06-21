---
title: jQuery学习笔记3：pagination的优化
date: 2017-02-02 09:58:12
tags:
- jQuery
categories:
- 技术
---
之前做的pagination功能，要从后端重新获得博客，再做前端显示，这就导致了一个bug：前端能得到的永远只有显示出来的10篇博客，搜索功能就只能在这十篇博客中做筛选。要想解决这个bug，首先肯定要能在前端获得所有的博客，所以我在主界面直接把所有博客全部加载了，然后把index大于等于10的博客隐藏，切换page的时候切换显示不同的博客。在搜索筛选完毕后，同样进行上述的分页操作，由于使用了两次，可以将这个分页方法单独提出来复用。

下面贴上代码。

```
//dividePage.js
function dividePage($elems){
	$elems.hide();
	$elems.filter(":lt(10)").show();
	var $pagination = $("#pagination");
	$pagination.find("li:eq(0)").click(function(){
		$elems.hide();
		$elems.filter(":lt(10)").show();
	});
	$pagination.find("li:gt(0)").click(function(){
		var index = $pagination.find("li").index(this);
		$elems.hide();
		$elems.filter(":gt('"+(index * 10 - 1)+"'):lt('10')").show();
	});
}
```

```
$(dividePage($("table tbody tr")))
//				修改pagination
				var $result = $blogs.filter(":visible");//获取所有搜索结果
				$result.filter(":lt(10)").hide();//隐藏后面页数的文章
				var resnum = $result.length;
				var pagenum = Math.ceil(resnum/10);
				var $pagination = $("#pagination");
				$pagination.empty();
				for(var i = 1; i <= pagenum; i++){
					$pagination.append("<li><a>" + i + "</a></li>");
				}
				dividePage($result);
```