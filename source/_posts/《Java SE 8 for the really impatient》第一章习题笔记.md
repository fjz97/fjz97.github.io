---
title: 《Java SE 8 for the really impatient》第一章习题笔记
date: 2017-02-09 16:47:33
tags:
- java
categories:
- 技术
---
2.

lambda表达式：

```
	public static File[] getChildDirectory(File file) {
		return file.listFiles(pathname -> {
			if(pathname.isDirectory()) {
				return true;
			}
			else {
				return false;
			}
		});
```

方法引用：

```
	public static File[] getChildDirectory(File file) {
		return file.listFiles(File::isDirectory);
	}
```

---

3.

```
	public static File[] getChildFilesByExt(File file, String ext) {
		return file.listFiles((dir, name) -> {
			if (name.substring(name.lastIndexOf(".") + 1).equals(ext)) {
				return true;
			} else {
				return false;
			}
		});
	}
```

捕获file以及ext。

---

6.

```
public class Test {
	public static void main(String[] args) {
		
		new Thread(uncheck(() -> {
			System.out.println("start");
			Thread.sleep(1000);
			System.out.println("end");
		})).start();
	}

	public static Runnable uncheck(RunnableEx runnableEx) {
		return new Runnable() {
			@Override
			public void run() {
				try {
					runnableEx.run();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		};
	}
}

interface RunnableEx {
	public void run() throws Exception;
}
```

 uncheck实际上做的事就是，封装了一个Runnable并返回，其中Runnable的run方法调用了lambda表达式的代码段并检查了代码段。

我感觉是可以用`Callable<Void>`的，在代码段最后加上`return null`就可以了，确实可以跑起来。

---

7.

```
	public static void main(String[] args) {
		
		andThen(()->{System.out.println("1");}, ()->{System.out.println("2");});
		
	}
	
	public static Thread[] andThen(Runnable a, Runnable b) {
		Thread thread1 = new Thread(a);
		thread1.start();
		Thread thread2 = new Thread(b);
		thread2.start();
		Thread[] threads = new Thread[2];
		return threads;
	}
```

---

8.

```
    String[] names = { "Peter", "Paul", "Merry" };
    List<Runnable> runners = new ArrayList<Runnable>();
    for(String name : names) {
	    runners.add(() -> System.out.println(name));
    }
    for(Runnable r : runners) {
	    new Thread(r).start();
    }
```

合法

如果使用传统的for循环，不合法，编译器报错Local variable i defined in an enclosing scope must be final or effectively final。捕获的i值发生了变化，即为非法。与内部类相同，JDK1.8开始可以在内部类和lambda表达式中使用非final局部变量，前提是值不发生改变（本质上还是final的）。