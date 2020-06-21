---
title: 《Java SE 8 for the really impatient》第三章习题笔记
date: 2017-02-22 14:23:28
tags:
- java
categories:
- 技术
---
2.

```
	public static void main(String[] args) {
		ReentrantLock myLock = new ReentrantLock();
		withLock(myLock, ()->{
			System.out.println("running");
		});
	}
	
	public static void withLock(ReentrantLock myLock, Runnable runnable) {
		myLock.lock();
		try {
			runnable.run();
		} finally {
			myLock.unlock();
		}
	}
```

---

5.

```
	public static void main(String[] args) {
		Image image = new Image("C:/Users/Administrator/Desktop/1.jpg");
		Image newImage = transform(image, (x, y, colorAtXY) -> {
			if(x<10||x>image.getWidth()-10||y<10||y>image.getHeight()-10) {
				return Color.GREY;
			} else {
				return colorAtXY;
			}
		});
	}
	
	public static Image transform(Image in, ColorTransformer cf) {
		int width = (int) in.getWidth();
		int height = (int) in.getHeight();
		WritableImage out = new WritableImage(width, height);
		for(int x = 0; x < width; x++) {
			for(int y = 0; y < height; y++) {
				out.getPixelWriter().setColor(x, y, cf.apply(x, y, in.getPixelReader().getColor(x, y)));
			}
		}
		return out;
	}
```
