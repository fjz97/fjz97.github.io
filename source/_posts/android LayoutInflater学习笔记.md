---
title: android LayoutInflater学习笔记
date: 2017-03-25 21:33:47
tags:
- Android
categories:
- 技术
---
获取LayoutInflater
-----

有三种方法获取LayoutInflater，分别是：

```
LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

```
LayoutInflater inflater = LayoutInflater.from(context);
```

```
LayoutInflater inflater = getLayoutInflater();//在activity中使用
```

而后两种方法实际也是调用第一种方法，得到的是LayoutInflater的实现类PhoneLayoutInflater。

inflate方法
-----

在生成view的时候常常要使用inflate方法，在fragment的onCreateView方法中我习惯写

`View view = inflater.inflate(R.layout.xxx, container, false);`

在listview adapter的getView方法中我又习惯写

`View view = inflater.inflate(R.layout.xxx, null);`

而却完全不明白这两个重载之间的区别。实际上inflate方法有四个重载而前三个方法本质也是调用最后一个方法。

![inflate](/images/19/inflate.png)

那么回到上面的两个例子，它们究竟有什么不同呢。通过下面这个例子解释一下。

布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:text="hello world"
    android:background="@android:color/darker_gray"
    android:gravity="center"/>
```

```
	<Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="加载布局"/>

    <LinearLayout
        android:id="@+id/root"
        android:orientation="vertical"
        android:layout_width="300dp"
        android:layout_height="300dp">
    </LinearLayout>
```

MainActivity

```
final LinearLayout root = (LinearLayout) findViewById(R.id.root);
        Button btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                LayoutInflater inflater = getLayoutInflater();
                View view = inflater.inflate(R.layout.item, null);
                root.addView(view);
            }
        });
```

我们给ViewGroup传null，运行结果如下

![ViewGroup_null](/images/19/ViewGroup_null.png)

发现我们给textview设定的match_parent属性并没有生效（毕竟它没有parent）。这就是第二个参数的作用所在，给生成的view限定一个parent，修改代码如下

`View view = inflater.inflate(R.layout.item, root, true);`

运行结果则让人满意

![ViewGroup_root](/images/19/ViewGroup_root.png)

另外第三个Boolean参数的作用是决定是否将view加入parent的xml文档结构里去，默认为true，当为false时，parent只作为参照决定view的位置和大小，而不把view加入它的xml文档里去，当然，这样的话它不存在与文档结构中，所以也不会显示出来。