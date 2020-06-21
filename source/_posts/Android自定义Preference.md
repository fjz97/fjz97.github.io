---
title: Android自定义Preference
date: 2018-05-21 17:51:11
tags:
- Android
categories:
- 技术
---
在写设置时参考了EH，发现使用了Preference类，它帮我们自动实现了ListView，只需要编写一个xml文件就可以实现各种样式的设置项，并自动将设置项保存到SharedPreferences内，非常方便。具体可以参考这篇文章：[Android Preference 设置偏好全攻略](https://www.cnblogs.com/valenhua/archive/2017/10/03/7624640.html)

这篇文章讲一下如何重写Preference，以实现自定义样式。

<!-- more -->

---

在开始前
---

我们需要一个颜色设置选项，可以选择不同的颜色，并把选择的颜色显示在右侧。

那么理所应当的，我们需要写一个布局文件：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="60dp"
                android:gravity="center_vertical" >

    <TextView
            android:id="@+id/title"
            android:text="color"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dp" />

    <ImageView
            android:id="@+id/color"
            android:layout_width="20dp"
            android:layout_height="20dp"
            android:layout_alignParentRight="true"
            android:layout_alignTop="@+id/title"
            android:layout_marginRight="30dp"
            android:background="@color/colorPrimary" />

</RelativeLayout>
```

just like this:

![layout](/images/27/layout.png)

实现
===

接下来重写Preference，我们看到重载方法中有onCreateView，那么显而易见，我们应该在这里inflate我们的布局文件。

```
public class ColorPreference extends ListPreference {
	@Override
    protected View onCreateView(ViewGroup parent) {
        super.onCreateView(parent);
        View view = LayoutInflater.from(getContext()).inflate(R.layout.preference_color, parent, false);
        return view;
    }
}
```

这样，我们就可以在preference的xml文件里使用我们自定义的Preference了：
```
<com.fengjingzhe.fightpicture.preference.ColorPreference
            android:key="theme"
            android:entries="@array/settings_theme_colors"
            android:entryValues="@array/settings_theme_colors_values"/>
```

效果如图

![color_preference](/images/27/color_preference.png)

One More Step
===

虽然我们实现了一个有着自定义View的Preference，但是TextView和ImageView中的内容都被限制了，需要每次到layout中更改，也不能动态设置内容，可重用性很差，我们希望实现的是一个通用的颜色选项，那么就需要做一点修改。

回想起之前使用fresco的经验，当我们需要给SimpleDraweeView加上一个url或是设置默认图案的时候，我们在xml文档中使用了fresco的自定义属性，那么在这里，我们肯定也能使用自定义属性。

在attrs.xml中添加我们的自定义属性title和imageColor，分别对应了TextView和ImageView的内容。

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="fightpicture">
        <attr name="title" format="string"></attr>
        <attr name="imageColor" format="string"></attr>
    </declare-styleable>
</resources>
```

接着我们在Preference的xml文档中使用我们的自定义属性：

```
        <com.fengjingzhe.fightpicture.preference.ColorPreference
            android:key="theme"
            fightpicture:title="@string/settings_theme_color"
            fightpicture:imageColor="?attr/colorPrimary"
            android:entries="@array/settings_theme_colors"
            android:entryValues="@array/settings_theme_colors_values" />
```

接下来的思路很明确，我们要在自定义的Preference类中获取到这两个属性并设置到TextView和ImageView上。

```
public class ColorPreference extends ListPreference {

    private String title;
    private String color;

    public ColorPreference(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public ColorPreference(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.fightpicture, defStyle, 0);
        title = a.getString(R.styleable.fightpicture_title);
        color = a.getString(R.styleable.fightpicture_imageColor);
        a.recycle();
    }

    @Override
    protected View onCreateView(ViewGroup parent) {
        super.onCreateView(parent);
        View view = LayoutInflater.from(getContext()).inflate(R.layout.preference_color, parent, false);
        if (title != null) {
            TextView titleTextView = view.findViewById(R.id.title);
            titleTextView.setText(title);
        }
        if (color != null) {
            ImageView colorImageView = view.findViewById(R.id.color);
            colorImageView.setBackgroundColor(Color.parseColor(color));
        }
        return view;
    }
}
```

在构造方法中使用TypedArray可以获取到xml文档中使用的属性，接下来拿到子View并将属性值设置上去就大功告成了！

再接着回想fresco，我们在给它设置url属性的时候，往往都不是在xml文档中，而是在代码中，也就是说，我们的自定义Preference还得具备动态设置属性的功能，很简单，我们给类添加两个set方法。

```
    public void setTitle(String title) {
        this.title = title;
    }

    public void setColor(String color) {
        this.color = color;
    }
```

由于PreferenceFragment的onCreate回调肯定先与Preference的onCreateView回调，我们在PreferenceFragment的onCreate回调中拿到Preference实例并对它设置属性就可以了。

```
ColorPreference theme = (ColorPreference) findPreference(THEME_KEY);
theme.setTitle(title);
theme.setColor(color);
```