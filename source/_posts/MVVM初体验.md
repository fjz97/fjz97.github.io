---
title: MVVM初体验
date: 2019-05-18 00:31:29
tags:
- Android
categories:
- 技术
---
### 前言
网上关于MVVM的文章很多了，本文站在一个初学者的角度，从实践出发，讲一下我对MVVM的理解和体验，如有误，待勘验。

---

### 为什么要使用MVVM
为什么要使用MVVM？最关键的，就是解耦，MVVM将项目分成View、ViewModel、Model三层，各层完成不同的工作，使逻辑更加独立清晰，大大方便了维护和测试。MVVM与MVP有何不同？
![MVP](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvp.png)![MVVM](http://zjutkz.net/images/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC-MVP%E5%92%8CMVVM/mvvm.png)
看图，MVP的View和Presenter双向依赖，而MVVM的View和ViewModel是单向依赖的，ViewModel持有View，View却不持有ViewModel，更进一步降低了耦合。如何实现View和ViewModel的单向依赖？有两种方法，一种是通过Google官方的DataBinding机制，这是一种数据驱动的思想，类似Vue，另一种是使用RxJava，这是一种事件驱动的思想，类似RxBus，本文使用RxJava作为解决方案。

---

### 开始之前
在着手代码前，我有必要先讲一下RxJava。这是我第一次使用RxJava，其实直到现在，我对于它还是一知半解，我希望用通俗的语言介绍一下它，尽管很可能不正确。
首先，要知道的是，RxJava最核心的思想是异步，实现的方式是观察者模式，Observable类对应了被观察者，Observer类对应了观察者。当被观察者和观察者被创建后，调用Observable.subscribe(Observer)即完成订阅，订阅即触发，Observable开始执行耗时操作，当操作完成后，通知Observer进行回调。这是最基本的异步使用方法，即使不用RxJava，我们也可以用Message-Handler去实现。但我不想用这种角度去看待Observable和Observer，在我看来，Observable是数据源，Observer是数据消费者。
接下来，要说的是Subject类，Subject有点特殊，它既是Observable，又是Observer，也就是说，它既可以生产数据，又可以消费数据。
最后再简单提一下CompositeDisposable这个类，Observable在调用subscribe方法后，会返回一个Disposable类，用于取消订阅，CompositeDisposable顾名思义可以对Disposable进行统一管理，当想要取消订阅的时候，调用CompositeDisposable.clear()就可以了。
以上就是项目中用到的所有RxJava提供的类。

---

### 实现
直接上手代码，本文代码完全参考了[这个项目](https://github.com/onlynight/V2EX)，为了使代码更加简洁，我只创建了三个类，分别对应了View、ViewModel、Model，并省去了一些生命周期方法，在实际的开发中，需要做一定的封装，并做好生命周期控制。

先看Model类，Model类比较简单，主要负责从网络或数据库获取数据，这里模拟一下从网络获取数据，返回一个Observable类。
```
public class Model {
    //模拟从服务器获取数据
    public Observable<List<String>> getData() {
        return Observable.create((emitter) -> {
            Thread.sleep(2000);
            List<String> list = new ArrayList<>();
            list.add(String.valueOf(System.currentTimeMillis()));
            emitter.onNext(list);
        });
    }
}
```
接下来是ViewModel类，ViewModel类持有一个Subject，作为数据消费者，通过bindData方法，将其与Model获取到的数据进行绑定。
```
public class ViewModel {

    private CompositeDisposable compositeDisposable = new CompositeDisposable();

    private Model model = new Model();

    private PublishSubject<List<String>> data = PublishSubject.create();

    private <T> void bindData(Observable<List<T>> observable, PublishSubject<List<T>> subject) {
        compositeDisposable.add(observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subject::onNext));
    }

    public void getUpdateData() {
        bindData(model.getData(), data);
    }

    public PublishSubject<List<String>> getData() {
        return data;
    }
}
```
最后是View，View也有bindData方法，它将ViewModel中获得到的Subject（这里成为了数据源）与一个消费方法绑定，这样，每当数据源发生改变了后，消费方法就会被调用。这里值得一提的是，View和ViewModel都有bindData方法，它们在实现上是一样的，具体的效果却有所不同，View的bindData方法确实如同名字一样，将生产者和消费者进行了绑定，但是ViewModel的bindData只是让Subject获取了数据，并没有进行绑定，因为Model每次生产的数据源引用的不是同一个对象。
```
public class MainActivity extends AppCompatActivity {

    private ListViewAdapter adapter;

    private CompositeDisposable compositeDisposable = new CompositeDisposable();

    private ViewModel viewModel = new ViewModel();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.button);
        ListView listView = findViewById(R.id.list_view);

        adapter = new ListViewAdapter(this);
        listView.setAdapter(adapter);

        bindData(viewModel.getData(), this::updateData);
        button.setOnClickListener((v) -> {
            viewModel.getUpdateData();
        });
    }

    private <T> void bindData(Subject<T> subject, Consumer<? super T> consumer) {
        compositeDisposable.add(subject.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(consumer));
    }

    private void updateData(List<String> data) {
        swipeRefreshLayout.setRefreshing(false);
        adapter.setData(data);
        adapter.notifyDataSetChanged();
    }
}
```
综上，我们是通过这种观察者模式实现的View与ViewModel解耦，实现的关键在于Subject，它可以同时作为数据的生产者和消费者，它存在于ViewModel中，一方面从Model中获取数据，一方面当数据变化时通知View进行操作。
[源码](https://github.com/fjz97/MVVMDemo)
