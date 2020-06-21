---
title: Bootstrap-table制作功能强大的网页表格
date: 2017-05-17 20:08:52
tags:
- javascript
categories:
- 技术
---
前言
===

最近在做数据库课程设计，想了想swing那low到爆的界面，决定还是做网页。其实需求无非就是将数据库数据前端展示一下，直接用bootstrap的表格却显得不好看，灵机一动参考了poi-statics的表格，发现它使用了一款表格插件，就是Bootstrap-table，试了一下后发现，真的非常强大！！！

Bootstrap-table官方文档：http://bootstrap-table.wenzhixin.net.cn/zh-cn/documentation/

---

刚开始看poi-statics的源码，因为我知道它是用的Bootstrap，以为仿照了写就可以实现一样的效果，结果一看代码密密麻麻，不由感叹：连写wiki的人水平™怎么都这么高？？再一看CSS引入，Bootstrap-table.css？？瞬间发现猫腻~加入了Bootstrap-table的js、css的CDN，当然，引入Bootstrap也是必要的（此外还需要jQuery），之后对table启用插件就可以使用了，代码全由插件生成，就很舒服了~

启用插件有两种方式，文档里写的很清楚，一是通过js启动，好处是维护性和扩展性比较好，也就是解耦了。第二张比较low的方法是直接通过设置table的属性启用，我这种菜鸟当然选择第二种啦！要配置表格属性我们就得熟悉它的属性，我直接贴一段Demo中的代码看看我使用了哪些属性：

```
<table id="tableSal" data-toggle="table" data-url="sale" data-method="post"
    			data-toolbar="#toolbarSal"
 				data-pagination="true"
 				data-search="true"
 				data-show-refresh="true"
 				data-show-toggle="true"
 				data-show-columns="true"
 				data-page-size="10">
    			<thead><tr>
    			<th data-checkbox="true"></th>
    			<th data-field="id" data-sortable="true">订单号</th>
    			<th data-field="date" data-sortable="true">销售日期</th>
    			<th data-field="type">轿车型号</th>
    			<th data-field="color">颜色</th>
    			<th data-field="number" data-sortable="true">数量</th>
    			<th data-field="customer">客户</th>
    			<th data-field="employee">负责人</th>
    			</tr></thead>
</table>
```

`data-toggle="table"` 不用写 JavaScript 直接启用表格。想要用属性配置必须得设置这个。

`data-url="sale"` 服务器数据的加载地址。服务器返回必须是json。

顺便看下我servlet是怎么写的：

```
		response.setContentType("text/javascript;charset=utf-8");
		response.setCharacterEncoding("UTF-8");
		response.getWriter().write(array.toString());
```

`data-toolbar="#toolbarSal"` 一个jQuery 选择器，指明自定义的toolbar。toolbar会出现在表格的左上方，另外我试过用class来选择，没有效果。

`data-pagination="true"` 设置为true会在表格底部显示分页条。

`data-search="true"` / `data-show-refresh="true"` / `data-show-toggle="true"` / `data-show-columns="true"` 分别对应了右上方的搜索栏和刷新、切换显示模式、隐藏列按钮。

`data-page-size="10"` 设置一页显示的数据。

除了表格属性，还有列属性：

`data-checkbox="true"` 设置一列为复选框

`data-field="number"` 设置一列的field，与服务器返回的json数据键对应，另外，在js中得到的行对象可以通过row.number得到行对应的number列的值。

`data-sortable="true"` 设置该列可排序，对于一般的数字类型值就不用再额外增加排序规则，如果需要另外增加排序规则，可以通过属性data-sorter来添加排序规则函数。

---

接下来看下它的方法，使用方法的语法：`$('#table').bootstrapTable('method', parameter)`。

我们以删除为例。

```
var id = $.map($("#tableSal").bootstrapTable('getSelections'), function(row){ return row.id; });
$("#tableSal").bootstrapTable('remove', {
					field: 'id',
					values: id
				});
```

getSelections方法得到选中的行对象数组，我们用jQuery的map方法得到它们的id。

remove方法删除表中的数据，它接受两个参数，field: 需要删除的行的 field 名称。values: 需要删除的行的值，类型为数组。这里的删除当然只是对表格的数据进行删除，对后端是没有影响的，刷新一下数据又回来了，所以方法叫remove不叫delete~如果想操作后端的话，用ajax就行了。

暂时我就只用到了这些属性和方法，除此以外，Bootstrap-table还具有事件监听，如行点击事件等，另外，我看到poi-statics上还有展开数据等功能，真的是非常全面强大！我用到的只是冰山一角，以后再有用到的机会再做记录吧~