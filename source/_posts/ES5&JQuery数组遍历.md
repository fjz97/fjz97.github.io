---
title: ES5&JQuery数组遍历
date: 2017-12-07 16:30:51
tags:
- jQuery
- javascript
categories:
- 技术
---
ES5 forEach

1.不支持IE9之前的版本。
2.匿名函数中的this都是指Window。
3.只能遍历数组。
4.没有返回值。

```
var ary = [12,23,24,42,1]; 
var res = ary.forEach(function (value,index,array) { 
array[index] = value*10; 
}) 
console.log(res);//--> undefined; 
console.log(ary);//--> 通过数组索引改变了原数组
```





ES5 map

1.不支持IE9之前的版本。
2.匿名函数中的this都是指Window。
3.只能遍历数组。
4.返回一个新的数组，原数组不变。map的回调函数中支持return返回值；return的是啥，相当于把数组中的这一项变为啥。

```
var ary = [12,23,24,42,1]; 
var res = ary.map(function (value,index,array) { 
return item*10; 
}) 
console.log(res);//-->[120,230,240,420,10];原数组拷贝了一份，并进行了修改
console.log(ary);//-->[12,23,24,42,1];原数组并未发生变化
```





JQuery $.each

1.与ES5 forEach类似，但是第一二两个参数相反。
2.支持IE9之前版本。
3.能遍历数组和类。

```
$.each( ["a","b","c"], function(i, v){ 
alert( i + ": " + v ); 
}); 

$.each( { name: "John", lang: "JS" }, function(k, v){ 
alert( "Name: " + k + ", Value: " + v ); 
}); 
```





JQuery $.map

1.与ES5 map类似，但是第一二两个参数相反。
2.支持IE9之前版本。
3.能遍历数组和类。

```
var arr=$.map( [0,1,2], function(v){ 
return v + 4; 
}); 

$.map({"name":"Jim","age":17},function(k, v){ 
console.log( k+":"+v ); 
}); 
```

再纠正个认知错误：each方法是JQuery对象的方法，对JQuery对象数组进行遍历，this指向该DOM对象，同理也有JQuery对象的map方法。