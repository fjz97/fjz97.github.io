---
title: android Toolbar学习笔记
date: 2017-03-26 20:12:27
tags:
- Android
categories:
- 技术
---
学习Toolbar之前，需要了解Toolbar与ActionBar之间的区别。这里贴上官方文档对Toolbar的解释。

>A Toolbar is a generalization of action bars for use within application layouts. While an action bar is traditionally part of an Activity's opaque window decor controlled by the framework, a Toolbar may be placed at any arbitrary level of nesting within a view hierarchy. An application may choose to designate a Toolbar as the action bar for an Activity using the setSupportActionBar() method.

简单翻译一下，Toolbar是ActionBar的更一般化，他可以存在于一个view的任何层次，我们可以给Activity定制一个Toolbar用于取代ActionBar。

究其历史原因，Android3.0推了ActionBar这个控件，而到了2013年Google开始大力地推动所谓的android style，想要逐渐改善过去纷乱的界面设计，希望让终端使用者尽可能在android手机有个一致的操作体验，Toolbar的出现意味着官方在某些程度上认为ActionBar限制了开发与设计的弹性。

（2018/5/10修订：Activity不带ActionBar，可以给它添加ActionBar而不能给它添加Toolbar，这里的添加是指代码中setSupportActionBar，而不是布局文件中添加，如果强行在布局文件中添加，menu不会显示。AppCompatActivity带ActionBar，也可以给它添加Toolbar，如果AppCompatActivity不隐藏ActionBar而添加Toolbar，会报运行时异常）
由于Activity是自带ActionBar的，所以想要使用Toolbar，先得将ActionBar隐藏起来。我们可以在style.xml中更改Activity的Theme，加入下面两行代码。

```
        <item name="windowNoTitle">true</item>
        <item name="windowActionBar">false</item>
```

或者更方便的，我们让Theme的parent为Theme.AppCompat.Light.NoActionBar

接着在Activity的Layout里面加入我们Toolbar的xml文件。

```
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
         android:id="@+id/toolbar"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         <!-- 当ToolBar没有添加任何东西时给其一个最小高度 -->
         android:minHeight="?attr/actionBarSize"
         android:background="?attr/colorPrimary">

</android.support.v7.widget.Toolbar>
```

现在来看看效果，Toolbar已经成功显示了。

![toolbar](/images/20/toolbar1.png)

得到了Toolbar以后，我们接下来研究如何给Toolbar自定义颜色和增加控件。

<!-- more -->

1.自定义颜色
-----

通过这张图片，我们来了解一下颜色的修改。

![toolbar_color](/images/20/toolbar_color.png)

1.状态栏底色：通过在Theme里设定colorPrimaryDark

2.ToolBar底色：通过在Toolbar控件里设置background属性（如果是ActionBar，则在Theme里设定colorPrimary，我们往往设定ToolBar的时候沿用这个属性）

3.导航栏底色：通过在Theme（v21）里设定android:navigationBarColor属性

4.windowBackground：通过在Theme里设定android:windowBackground属性

2.控件
-----

查阅官方文档，Toolbar提供了这些控件。

> - A navigation button. This may be an Up arrow, navigation menu toggle, close, collapse, done or another glyph of the app's choosing. This button should always be used to access other navigational destinations within the container of the Toolbar and its signified content or otherwise leave the current context signified by the Toolbar. The navigation button is vertically aligned within the Toolbar's minimum height, if set.
> - A branded logo image. This may extend to the height of the bar and can be arbitrarily wide.
> - A title and subtitle. The title should be a signpost for the Toolbar's current position in the navigation hierarchy and the content contained there. The subtitle, if present should indicate any extended information about the current content. If an app uses a logo image it should strongly consider omitting a title and subtitle.
> - One or more custom views. The application may add arbitrary child views to the Toolbar. They will appear at this position within the layout. If a child view's Toolbar.LayoutParams indicates a Gravity value of CENTER_HORIZONTAL the view will attempt to center within the available space remaining in the Toolbar after all other elements have been measured.
> - An action menu. The menu of actions will pin to the end of the Toolbar offering a few frequent, important or typical actions along with an optional overflow menu for additional actions. Action buttons are vertically aligned within the Toolbar's minimum height, if set.

结合这张图，对这些控件的布局应该就一目了然了，下面上Demo。

![widget](/images/20/widget.png)

首先获取到Toolbar并通过setSupportActionBar方法给Activity设置Toolbar。

```
        Toolbar mToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolbar);
```
        
对上图1234的设置比较简单，直接按照上述方法进行设置。

```
        mToolbar.setTitle("Demo");
        mToolbar.setSubtitle("jzfeng");
        mToolbar.setLogo(R.drawable.icon);
        mToolbar.setNavigationIcon(R.drawable.icon);
        mToolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                MainActivity.this.finish();
            }
        });
```

而对于菜单的设置，Toolbar与ActionBar并无差异，同样是在res/menu下写菜单文件xml，然后在Activity的onCreateOptionsMenu中inflate菜单view。

```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      tools:context=".MainActivity">
    <item
        android:id="@+id/action_settings"
        android:title="设置"
        app:showAsAction="never"/>

    <item
        android:id="@+id/action_test"
        android:title="测试"
        android:icon="@drawable/icon"
        app:showAsAction="ifRoom"/>
</menu>
```

```
        mToolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                if(item.getItemId() == R.id.action_settings) {
                    Toast.makeText(MainActivity.this, "settings clicked", Toast.LENGTH_SHORT).show();
                } else if(item.getItemId() == R.id.action_test) {
                    Toast.makeText(MainActivity.this, "test clicked", Toast.LENGTH_SHORT).show();
                }
                return true;
            }
        });
```

```
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu, menu);
        return true;
    }
```

注意listener的设定必须写在setSupportActionBar方法之后，否则不起效，另外，optionsMenu点击事件也会回调到Activity的onOptionsItemSelected方法中，所以写在那里也没问题。

现在看看效果吧。

![toolbar](/images/20/toolbar2.png)

哇，丑爆了。。。事实上，我们这种做法确实完全没有发挥出Toolbar的真实威力。Toolbar既然存在于布局文件中，那么我们当然可以在布局文件中直接给它添加view。

3.添加子View
-----

明白了该怎这么做，我直接附上代码。

```
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
         android:id="@+id/toolbar"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:minHeight="?attr/actionBarSize"
         android:background="?attr/colorPrimary">

    <ImageView
        android:id="@+id/nav_icon"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:layout_margin="3dp"
        android:src="@drawable/icon"/>

    <ImageView
        android:id="@+id/logo"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginLeft="20dp"
        android:src="@drawable/icon"/>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="25dp"
            android:textSize="20sp"
            android:text="Demo"/>

        <TextView
            android:layout_marginTop="3dp"
            android:layout_width="wrap_content"
            android:layout_height="20dp"
            android:textSize="14sp"
            android:text="jzfeng"/>

    </LinearLayout>

</android.support.v7.widget.Toolbar>
```

```
        getSupportActionBar().setDisplayShowTitleEnabled(false);
        mToolbar.findViewById(R.id.nav_icon).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                MainActivity.this.finish();
            }
        });
```

注意这里要使用getSupportActionBar().setDisplayShowTitleEnabled(false)使Toolbar不再默认显示标题。这之后我们就可以想获取到哪个View就获取到哪个View，想进行什么操作就进行什么操作了。最后附上完成图~

![toolbar](/images/20/toolbar3.png)