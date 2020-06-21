---
title: Android性能优化之ViewHolder&convertView
date: 2018-03-31 21:42:30
tags:
- Android
categories:
- 技术
---
查阅了许多博客，对ViewHolder的优化原理仍然不明白，今天深入研究了一下，总算是理解了。
先贴上项目里使用的一个图片GridView的Adapter代码：

```
public class MyAdapter extends BaseAdapter {

    private LayoutInflater mInflater;
    private Context context;
    private List<String> pictureUrls;

    public MyAdapter(Context context, List<String> pictures) {
        mInflater = LayoutInflater.from(context);
        this.context = context;
        this.pictureUrls = pictures;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder = null;
        if (convertView == null) {
            convertView = mInflater.inflate(R.layout.picture_item_layout, parent, false);
            viewHolder = new ViewHolder();
            viewHolder.picture = convertView.findViewById(R.id.picture);
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }
        viewHolder.picture.setImageURI(getItem(position));
        return convertView;
    }

    @Override
    public int getCount() {
        return pictureUrls.size();
    }

    @Override
    public String getItem(int position) {
        return pictureUrls.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    private final class ViewHolder {
        SimpleDraweeView picture;
    }
}
```

其他三个方法不用多说，关键在于getView方法。
getView方法用于返回item的View，我们看方法的参数convertView，通过查阅博客，我知道了convertView是android中Recycler构件缓存的View，当有item移出屏幕的时候，获取下一个item时调用的getView方法的convertView参数就是被移除的item的View，如下图：

![Recycler](/images/5/Recycler.jpg)

至此都不难理解，这些预备知识我早先就知道了，却仍然不明白如何实现的性能优化。

苦思冥想后，恍然大悟，我不过是进入了一次思维误区：
从来没见过ViewHolder这种复用机制的我感觉太新鲜了，认为ViewHolder才是主角，convertView不过是个参数，小小配角而已，殊不知android如此大费周章编写的构件，怎么可能会给一个甚至没有加入SDK的自定义类抢去风头！实现性能优化的恰恰是convertView，ViewHolder不过是个添头。

不论是GridView还是ListView，它们的item都是从同一个layout中inflate的，所以说白了，我们接下来要生成的View，从组成上说，与convertView是一样的，唯一不同的就是其中填充的数据，复用convertView，就可以省下重新inflate的开销（这可是IO操作），达到提升性能的效果。

那么ViewHolder呢？它有什么用呢？要想解决这个问题，先回答另一个问题，ViewHolder中保存的是什么？以上面的代码为例，是SimpleDraweeView，也就是新View和convertView存在内容上不同的子控件，在getView方法里我们肯定要修改它的内容，那就肯定要获取到它，使用传统的findViewById也存在一定的时间开销，当要修改内容的控件增多时，开销会更大，所以使用ViewHolder通过View的setTag方法把控件保存起来，需要修改的时候再取出来，可以省下一定的开销。也就是说，这是一种空间换时间的策略。

至此我们的问题解决了，不妨跟着例子的代码走一遍：

 1. GridView初始化，创建item，调用getView方法，此时convertView为空，我们通过LayoutInflater填充View赋给convertView，并创建ViewHolder绑定到View上（ViewHolder里面有我们获取到的SimpleDraweeView子控件），接着给SimpleDraweeView设置图片的URL（修改子控件内容），最后返回convertView。这个阶段会重复若干次，直到屏幕显示满图片。
 2. 下拉屏幕，获取新的item，此时创建item，调用getView方法，convertView就不为空了，我们直接拿到ViewHolder并修改其中子控件的内容，此时这个被回收的convertView就摇身一变，变成了我们期望的新View了（图片URL被改变了），于是我们将它返回。

最后满足一下我的好奇心~当GridView多个View被回收，又创建多个新View的时候，convertView会是怎么样对应的呢？
我们对上述例子代码修改SimpleDraweeView内容的一行注释掉，看看运行结果~

![before](/images/5/before.png)

![after](/images/5/after.png)

可以看出来，当下拉加载第一排图片的时候，第一张图片的convertView是下拉前的第一张图片，也就是第一个被回收的图片，第二三四张图片的convertView却是null，所以加载了正确的图片。接着下拉加载第二排的图片，第一二张图片的convertView是下拉前的第三四张图片，也就是最后回收的两张图片。