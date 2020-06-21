---
title: RecyclerView学习笔记
date: 2017-04-07 22:14:38
tags:
- Android
categories:
- 技术
---
以前在做app时，我习惯使用ListView、GridView等控件来实现列表、网格展示数据的效果。今天研究b站客户端源码的时候，稍微了解了一下RecyclerView控件，它可以用来替代以往的ListView和GridView，并且对上拉刷新和下拉加载功能有很方便的回调，此外添加删除操作也有特殊的动效，而且结合前面所学的Design Library，CoordinatorLayout对它有很好的支持，看来很值得学习一下它。

学习参考：[【FastDev4Android框架开发】RecyclerView完全解析,让你从此爱上它(二十八)](https://blog.csdn.net/developer_jiangqq/article/details/49927631)

[【FastDev4Android框架开发】RecyclerView完全解析之下拉刷新与上拉加载SwipeRefreshLayout(三十一)](https://blog.csdn.net/developer_jiangqq/article/details/49992269)

<!-- more -->

---

1.基本使用
===

引入RecyclerView后，设置LayoutManager和Adapter就可以使用了。

```
        RecyclerView recyclerView = (RecyclerView) view.findViewById(R.id.recycler_view);
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(getContext());
        linearLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
        recyclerView.setLayoutManager(linearLayoutManager);
        recyclerView.setAdapter(new RecyclerViewAdapter(this.getContext()));
```

Adapter的设置和ListView不太一样。

```
//Adapter必须继承自RecyclerView.Adapter，MyViewHolder必须继承自RecyclerView.ViewHolder
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.MyViewHolder> {
    public RecyclerViewAdapter(Context context) {
        titles = new String[20];
        mInflater = LayoutInflater.from(context);
        for (int i = 0; i < 20; i++) {
            titles[i] = "item" + i;
        }
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = mInflater.inflate(R.layout.item_recycler, parent, false);
        MyViewHolder viewHolder = new MyViewHolder(view);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        holder.textView.setText(titles[position]);
    }

    @Override
    public int getItemCount() {
        return titles.length;
    }

    public class MyViewHolder extends RecyclerView.ViewHolder {
        TextView textView;
        public MyViewHolder(View view) {
            super(view);
            textView = (TextView) view.findViewById(R.id.text_view);
        }
    }

    private String[] titles;
    private LayoutInflater mInflater;
}
```

这里不做太多的解释，仿照着写就可以，值得注意的地方是泛型中的ViewHolder类型必须与onCreateViewHolder的返回类型保持一致。可以看出Google官方对ViewHolder的使用官方化了，正好以前看不懂，这下我也不用管它的内部实现了233。

<img src="/images/22/recyclerview.gif"  width="30%" height="30%">

和ListView没什么区别~通过改变LayoutManager的设置，我们还可以实现GridView的效果。

```
GridLayoutManager girdLayoutManager=new GridLayoutManager(getContext(),4);  
recyclerView.setLayoutManager(girdLayoutManager);  
```

<img src="/images/22/recyclerview_gridview.gif"  width="30%" height="30%">

2.点击事件监听
---

遗憾的是，RecyclerView并没有给我们提供点击事件的回调，我们需要自己实现回调的监听。

具体的思路就是在Adapter内添加一个监听点击事件的接口，开放一个set方法让外部重写这个函数式接口，在onBindViewHolder方法中获取到View对每个View添加OnClickListener，在里面调用接口的方法。

//接口

```
    public interface OnRecyclerItemClickListener {
        void onItemClick(View view, int position);
    }
```

//onBindViewHolder 

```
    public void onBindViewHolder(MyViewHolder holder, int position) {
        holder.textView.setText(titles[position]);
        if (onRecyclerItemClickListener != null)
            holder.itemView.setOnClickListener(v->onRecyclerItemClickListener.onItemClick(v, position));
    }
```

//外部调用 

```
        adapter.setOnItemClickListener((v, position) ->
            Snackbar.make(v, position + "", Snackbar.LENGTH_SHORT).show());
```

![click](/images/22/click.png)

3.添加/删除item
---

回想使用Listview的时候，我们对适配器内数据进行更改以后，使用notifyDatasetChange方法来刷新界面，在RecyclerView中，我们也可以使用notifyDatasetChange方法来刷新界面，但在这里，我们尝试使用更加高级的notifyItemInserted(position)和notifyItemDeleted(position)方法，这两个方法可以在任何位置添加或删除元素，并提供了动画效果。

先看个简单的删除例子

//main

```
        adapter.setOnItemClickListener((v, position) ->
            adapter.deleteItem(position));
```

//Adapter 

```
    public void deleteItem(int position) {
        titles.remove(position);
        notifyItemRemoved(position);
    }
```

//onBindViewHolder

```
        holder.itemView.setOnClickListener( v -> {
            int pos = holder.getLayoutPosition();
            onRecyclerItemClickListener.onItemClick(v, pos);
        });
```

在进行了添加删除操作后，由于Adapter更新不及时，position值就不一定准确了，所以应该使用ViewHolder.getLayoutPosition方法得到正确的pos值。（坑爹）

<img src="/images/22/remove.gif"  width="30%" height="30%">

添加操作我们放到上拉刷新里一起说。

4.上拉刷新
---

上拉刷新效果非常简单，只需要在RecyclerView外层嵌套一层SwipeRefreshLayout，然后在java代码中获取到它进行设置就行了。在我们完成刷新后记得要调用setRefreshing(false)使刷新动画结束。

```
		SwipeRefreshLayout swipeRefreshLayout findViewById(R.id.swipe_refresh_layout);
        swipeRefreshLayout.setOnRefreshListener(() -> {
            for (int i = 0; i < 5; i++) {
                adapter.addItem("new item" + i, 0);
            }
            swipeRefreshLayout.setRefreshing(false);
        });
```

```
	public void addItem(String title, int position) {
        titles.add(position, title);
        notifyItemInserted(position);
    }
```

<img src="/images/22/insert.gif"  width="30%" height="30%">

5.下拉加载
---

下拉刷新效果则是设置RecyclerView.setOnScrollListener，这里用到了LayoutManager.findLastVisibleItemPosition方法。

```
        recyclerView.setOnScrollListener(new RecyclerView.OnScrollListener() {
            private boolean isDown;
            private int lastVisibleItem;
            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == RecyclerView.SCROLL_STATE_IDLE && adapter.getItemCount() == lastVisibleItem + 1 && isDown) {
                    new Handler().postDelayed(()-> {
                        List<String> newTitles = new ArrayList<>();
                        for (int i = 0; i < 5; i++) {
                            newTitles.add("new item" + i);
                        }
                        adapter.addMoreItem(newTitles);
                    }, 2000);
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                if (dy > 0) {
                    isDown = true;
                } else {
                    isDown = false;
                }
                lastVisibleItem = linearLayoutManager.findLastVisibleItemPosition();
            }
        });
```

和上拉刷新一样，这样写并没有让新增的数据显示出来，用户甚至不知道自己进行了刷新操作。

<img src="/images/22/pull.gif"  width="30%" height="30%">

观察RecyclerView.Adapter.onCreateViewHolder方法，第二个参数是viewType，再看RecyclerView.Adapter的可重载方法，我们发现了getItemViewType方法，实际上在调用onCreateViewHolder方法前，会调用getItemViewType方法确定View的种类，onCreateViewHolder方法第二个参数是viewType就是这个方法的返回值。如果我们不重写这个方法，它会默认返回0。我们可以通过重写这个方法控制onCreateViewHolder方法生成不同的View。

```
    @Override
    public int getItemCount() {
        return titles.size() + 1;
    }
```

```
    @Override
    public int getItemViewType(int position) {
        if (position + 1 == getItemCount()) {
            return TYPE_FOOTER;
        } else {
            return TYPE_ITEM;
        }
    }
```

```
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (viewType == TYPE_ITEM) {
            View view = mInflater.inflate(R.layout.item_recycler, parent, false);
            return new MyViewHolder(view);
        } else if (viewType == TYPE_FOOTER) {
            View view = mInflater.inflate(R.layout.item_footer, parent, false);
           return new FooterViewHolder(view);
        }
        return null;
    }
```

当item为最后一个元素的时候，我们返回TYPE_FOOTER，生成footer view给用户提示数据正在加载。注意，当item过少而不满一页时，我们需要隐藏footer view。

<img src="/images/22/pull_footer.gif"  width="30%" height="30%">