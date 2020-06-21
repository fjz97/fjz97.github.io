---
title: Android Design Support Library（材料设计兼容库）学习
date: 2017-04-04 16:52:31
tags:
- Android
categories:
- 技术
---
android5.0引入了Material Design，后续发布的android design support library直接兼容到了android2.1，现在越来越多的App开始使用它的组件，这将是未来App的主流设计风格。

引入项目

`compile 'com.android.support:design:24.2.0'`

<!-- more -->

---

1.DrawerLayout&NavigationView
===

创建布局文件，官方推荐DrawerLayout作为根布局，添加NavigationView。

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.NavigationView
        android:id="@+id/navigation"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:headerLayout="@layout/nav_header"
        app:menu="@menu/layout_navigation_drawer">
    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout>
```

这里解释一下NavigationView里的几个重要属性。

android:layout_gravity：start/end，其中start表示view在左侧，end表示view在右侧。

app:headerLayout：表示view上部分的layout。

app:menu：表示view的菜单部分。

接下来创建headerLayout和menu布局文件。

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="320dp"
    android:layout_height="@dimen/nav_header_height"
    android:background="?attr/colorPrimary"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:theme="@style/ThemeOverlay.AppCompat.Dark"
    android:orientation="vertical"
    android:gravity="bottom">

</LinearLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_camera"
            android:icon="@android:drawable/ic_menu_camera"
            android:title="Import"/>
        <item
            android:id="@+id/nav_gallery"
            android:icon="@android:drawable/ic_menu_gallery"
            android:title="Gallery"/>
        <item
            android:id="@+id/nav_slideshow"
            android:icon="@android:drawable/ic_menu_slideshow"
            android:title="Slideshow"/>
        <item
            android:id="@+id/nav_manage"
            android:icon="@android:drawable/ic_menu_manage"
            android:title="Tools"/>
    </group>

    <item android:title="Communicate">
        <menu>
            <item
                android:id="@+id/nav_share"
                android:icon="@android:drawable/ic_menu_share"
                android:title="Share"/>
            <item
                android:id="@+id/nav_send"
                android:icon="@android:drawable/ic_menu_send"
                android:title="Send"/>
        </menu>
    </item>
</menu>
```

还差一步，我们需要监听返回键，当drawer被打开的时候可以通过返回键关闭它。

```
    public void onBackPressed() {
        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        if (drawer.isDrawerOpen(GravityCompat.START)) {
            drawer.closeDrawer(GravityCompat.START);
        } else {
            super.onBackPressed();
        }
    }
```

这样以后基本的效果就完成了。

![navigation](/images/21/navigation.png)

接下来看下如何设置NavigationView选项按钮的回调。这里的回调和之前我们学习的Toolbar上的menu回调很相似。既可以通过给NavigationView调用setNavigationItemSelectedListener方法来设置回调，也可以在Activity的onNavigationItemSelected方法里设置回调。具体可以看看对Android Menu的介绍（虽然navigation不属于三种menu）。

这里贴上通过Listener设置回调的代码。

```
NavigationView navigationView = (NavigationView) findViewById(R.id.navigation);
        navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                int selected = item.getItemId();
                if (selected == R.id.nav_share) {
                    Toast.makeText(MainActivity.this, "share clicked", Toast.LENGTH_SHORT).show();
                }
                return true;//返回true表示处理了事件
            }
        });
```

![navigation_click](/images/21/navigation_click.png)

2.DrawerLayout与Toolbar结合以及交互
===

首先要说的是，官方推荐让DrawerLayout作为根布局，NavigationView作为它的最后一个直接子view，如果不这么做，可能会有各种小bug，我已经体验过了。但是如果直接把Toolbar作为DrawerLayout的直接子view，Toolbar会占满整个屏幕。

我们注意到官方文档里有这样一句话：

> To use a DrawerLayout, position your primary content view as the first child with width and height of match_parent and no layout_gravity>.

也就是说，第一个子view会占满屏幕，所以我们应该把一个layout作为第一个子view并把它的layout_height设为match_parent，就像这样：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        
        <android.support.v7.widget.Toolbar
                xmlns:android="http://schemas.android.com/apk/res/android"
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:minHeight="?attr/actionBarSize"
                android:background="?attr/colorPrimary">

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
        
    </LinearLayout>

    <android.support.design.widget.NavigationView
            android:id="@+id/navigation"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            app:headerLayout="@layout/nav_header"
            app:menu="@menu/layout_navigation_drawer">
    </android.support.design.widget.NavigationView>

</android.support.v4.widget.DrawerLayout>
```

接着进入主题，DrawerLayout和Toolbar的交互。两者可以通过ActionBarDrawerToggle类来实现交互，我们来研究这个类。

ActionBarDrawerToggle本质上是DrawerLayout.DrawerListener接口的实现类，也就是说它是官方给我们提供的一个写好的DrawerListener，我们可以直接拿来用。

```
        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(this, drawerLayout, mToolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
        drawerLayout.setDrawerListener(toggle);
        toggle.syncState();
```

效果如图：

<img src="/images/21/toggle.gif" width="30%" height="30%">

那么我们想让NavigationView出现在Toolbar的下方呢？我思前想后也只能想到把Toolbar和DrawerLayout放在一个LinearLayout里，bug出现了，如图所示，点击空白处NavigationView收不回去。

<img src="/images/21/toggle_below.gif" width="30%" height="30%">

仔细想了好久，恍然大悟，还是把官方的话给忘了，明明就引用在上面。

**切记，DrawerLayout的第一个子view一定要给它分配个layout！！！**

也就是说，bug不是出在我们没有把DrawerLayout作为根布局，而是出在没给它加一个子layout（另外，这个子layout就算你给它的layout_height设为wrap_content而没给它放任何content，它的高度依然是全屏的）。改正后bug消除了。

<img src="/images/21/toggle_below_success.gif" width="30%" height="30%">

3.TextInputLayout&TextInputTextEdit
===

TextInputLayout&TextInputTextEdit对传统的EditText做了进一步的优化改善，赋予了它一些美观以及实用的动效。TextInputEditText继承自EditText，提供了不一样的UI，但是xml属性和方法是完全一致的。TextInputLayout继承自LinearLayout，使用非常简单，我们熟悉一下一些有用的属性和Java方法就可以了。

在布局里加入TextInputLayout&TextInputTextEdit，注意TextInputTextEdit必须存在于TextInputLayout里面，不然不会提供动效。

```
        <android.support.design.widget.TextInputLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/app_name">
            <android.support.design.widget.TextInputEditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
        </android.support.design.widget.TextInputLayout>
```

以上代码引入了最基本的TextInputLayout样式，注意，hint属性最好写在TextInputLayout中。

下面通过java代码对它进行一些操作。

1.错误提示。
---

我们来看一个邮箱检验的例子。

```
        String regex = "^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\\.[a-zA-Z0-9_-]+)+$";
        final Pattern pattern = Pattern.compile(regex);
        final TextInputLayout emailLayout = (TextInputLayout) findViewById(R.id.email_layout);
        TextInputEditText emailText = (TextInputEditText) findViewById(R.id.email_text);
        emailText.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                Matcher matcher = pattern.matcher(s);
                if(!matcher.matches()){
                    if(!emailLayout.isErrorEnabled()){
                        emailLayout.setErrorEnabled(true);
                    }
                    emailLayout.setError("邮箱格式不正确");
                } else {
                    emailLayout.setErrorEnabled(false);
                }
            }

            @Override
            public void afterTextChanged(Editable s) {

            }
        });
```

TextInputLayout.setErrorEnabled方法设置是否可以显示错误提示信息，对TextInputEditText设置监听，当内容改变时，如果邮箱不符合格式，那么允许显示错误提示信息并且通过setError方法显示它，反之则禁用错误提示信息。

<img src="/images/21/check_mail.gif" width="30%" height="30%">

4.FloatingActionButton
===

接下来来看个很简单的组件，FloatingActionButton，我们关注它的几个属性就好了。

```
<android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end|bottom"
            app:backgroundTint="#ff87ffeb"
            app:rippleColor="#33728dff"
            app:elevation="6dp"
            app:pressedTranslationZ="12dp"
            android:src="@android:drawable/ic_dialog_info"/>
```

backgroundTint：背景颜色，默认为Theme中的colorAccent

rippleColor：点击颜色，默认为Theme中的colorControlHighlight

elevation：阴影的大小

pressedTranslationZ：点击时的阴影大小

5.Snackbar
===

再来看一个简单的组件——Snackbar，它是一个升级版的Toast，与Toast不同的是，它可以提供一个feedback操作。

```
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //make方法第一个参数传的view我们一般就选取它提供的view（这里是fab）
                Snackbar.make(drawerLayout, "fab clicked", Snackbar.LENGTH_SHORT).setAction("DONE", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                }).show();
            }
        });
```

点击DONE或经过一小段时间后消失。

当然我们也可以设定自动消失的时间

```
Snackbar.make(drawerLayout, "fab clicked", Snackbar.LENGTH_SHORT).setDuration(5000).show();
```

6.TabLayout
===

1.基本使用
---

TabLayout提供了一个可以滑动的选项卡，我们在xml文件中引入它。

```
        <android.support.design.widget.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabMode="scrollable">
        </android.support.design.widget.TabLayout>
```

我们一般不会对TabLayout的样式做太多的定制，如果需要，移步：[TabLayout属性详解](https://www.jianshu.com/p/2b2bb6be83a8)

我们关注TabLayout最关键的属性app:tabMode，它有两个值，分别是scrollable和fixed。如果是scrollable，当屏幕显示不下所有tab的时候，有些tab会被隐藏；如果是fixed，所有tab会全部显示，当有很多个tab的时候，它们会被挤压到很小。

我们可以在xml文件中创建选项卡，但是一般习惯在java代码中创建。

`tabLayout.addTab(tabLayout.newTab().setText("tab1"));`

运行效果如图：

![tab](/images/21/tab.png)

我们发现TabLayout宽度并没有占满整个屏幕，这时候必须设置app:tabGravity属性为fill，而该属性只有tabMode为fixed的时候才生效。

```
        <android.support.design.widget.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabMode="fixed"
            app:tabGravity="fill">
        </android.support.design.widget.TabLayout>
```

![tab_fill](/images/21/tab_fill.png)

小结：当选项卡较少时，我们使用appMode:fixed配合tabGravity:fill；当选项卡较多时，我们使用appMode:scrollable。

2.TabLayout配合ViewPager使用
---

我们一般将TabLayout和ViewPager一起使用，实现像b站、知乎等app的效果。

只需要一行代码就可以实现它们之间的协作了

`tabLayout.setupWithViewPager(viewPager);`

本质上其实是调用了

`viewPager.addOnPageChangeListener(new TabLayout.TabLayoutOnPageChangeListener(tabLayout));`

还有，必须得实现ViewPager适配器中的getPageTitle方法，不然tab的标题无法正常显示。另外，我们就不需要自己创建tab了，会自动适配生成tab。实例代码如下：

```
		TabLayout tabLayout = findViewById(R.id.tab_layout);
        tabLayout.addTab(tabLayout.newTab());
        tabLayout.addTab(tabLayout.newTab());
        tabLayout.addTab(tabLayout.newTab());
        final List<String> titles = new ArrayList<>();
        titles.add("tab1");
        titles.add("tab2");
        titles.add("tab3");
        final List<LinearLayout> layouts = new ArrayList<>();
        layouts.add(new LinearLayout(this));
        layouts.add(new LinearLayout(this));
        layouts.add(new LinearLayout(this));
        ViewPager vp = findViewById(R.id.view_pager);
        vp.setAdapter(new PagerAdapter() {
            @Override
            public Object instantiateItem(ViewGroup container, int position) {
                container.addView(layouts.get(position));
                return layouts.get(position);
            }

            @Override
            public void destroyItem(ViewGroup container, int position, Object object) {
                container.removeView(layouts.get(position));
            }

            @Override
            public int getCount() {
                return 3;
            }

            @Override
            public boolean isViewFromObject(View view, Object object) {
                return false;
            }

            @Override
            public CharSequence getPageTitle(int position) {
                return titles.get(position);
            }
        });
        tabLayout.setupWithViewPager(vp);
```

<img src="/images/21/tablayout_viewpager.gif" width="30%" height="30%">

7.CoordinatorLayout（协调者布局）
===

CoordinatorLayout是一个强化版的FrameLayout，从名字上我们就可以知道它用于处理多个子view的协调互动，注意，子view必须是CoordinatorLayout的直接子view。

我们由几种效果的实现来了解它。

1.和FAB的互动
---

这个很简单，直接在CoordinatorLayout里引入FAB就行了。

2.Toolbar的向上滑动隐藏
---

将Toolbar用AppBarLayout包裹起来，并给Toolbar加上属性

`app:layout_scrollFlags="scroll|enterAlways"`

然后给要滑动的组件加上属性，这个组件一般是NestedScrollView（我使用ScrollView不奏效）。

`app:layout_behavior="@string/appbar_scrolling_view_behavior"`

<img src="/images/21/scroll.gif" width="30%" height="30%">

3.更一般的使用
---

[CoordinatorLayout的使用如此简单](https://blog.csdn.net/huachao1001/article/details/51554608)