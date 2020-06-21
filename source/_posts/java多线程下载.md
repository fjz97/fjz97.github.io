---
title: java多线程下载
date: 2018-04-28 17:30:28
tags:
- java
categories:
- 技术
---
在上一篇文章里，我们讨论了图片缓存速度慢的原因，发现IO并不是决定因素。那么问题出在哪呢？一篇关于java多线程下载的博客引起了我的注意：我们知道迅雷、旋风等下载器都是多线程下载的，会不会是单线程效率的问题影响了图片缓存速度呢？事不宜迟，我们开始验证。

----------

事前准备
=====

多线程下载的原理很简单，无非就是分段请求数据，最后再把每段数据拼凑起来，实现起来需要解决两个问题， 没错，就是怎么分段请求和怎么拼接数据。

1.HTTP/1.1 Header Range
-----

首先面临的问题是，怎么分段请求数据。
HTTP/1.1引入了一个新的header，名字叫做Range，它可以指定一个范围，然后http请求就会只请求那一段的数据，java代码可以这样写：

`conn.setRequestProperty("Range", "bytes=" + start + "-" + end);`

其中start就是数据的开始字节，end是数据的结束字节。

2.RandomAccessFile
-----

第二个问题是，怎么将这些分段的数据拼接起来。
这里我们需要用到java随机访问文件RandomAccessFile，它指定文件并通过seek方法跳转到该文件的任意字节处进行读写操作。于是我们的java代码可以这么写：

```
				raf = new RandomAccessFile(file, "rw");
				raf.seek(start);
				byte[] bytes = new byte[1024];
				int len;
				while ((len = is.read(bytes)) != -1) {
					raf.write(bytes, 0, len);
				}
```

为了方便测试，我写了一个java demo，代码如下：

```
public class Main {
	
	private static final String PICTURE_URL = "http://d1.music.126.net/dmusic/CloudMusic_official_5.1.0.573520.apk";
	private static final String FILE_NAME = "cloudmusic.apk";
	private static final int THREAD_NUM = 10;
	private static long startTime;
	private static long endTime;
	private static double usetime;
	private static int count = 0;
	
	public static void main(String[] args) {
		startTime = System.currentTimeMillis();
		try {
			File file = new File("C:/cdt/", FILE_NAME);
			URL url = new URL(PICTURE_URL);
			HttpURLConnection conn = (HttpURLConnection) url.openConnection();
			int size = conn.getContentLength();
			int piece = size % THREAD_NUM == 0 ? size / THREAD_NUM : (size / THREAD_NUM + 1);
			for (int j = 0; j < THREAD_NUM; j++) {
				int start = j * piece;
				int end = (j + 1) * piece - 1;
				if (j == THREAD_NUM - 1) end = size;
				new DownloadThread(file, url, start, end).start();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	private static class DownloadThread extends Thread {
		private File file;
		private URL url;
		private int start;
		private int end;
		
		public DownloadThread(File file, URL url, int start, int end) {
			this.file = file;
			this.url = url;
			this.start = start;
			this.end = end;
		}
		
		public void run() {
			InputStream is = null;
			RandomAccessFile raf = null;
			try {
				HttpURLConnection conn = (HttpURLConnection) url.openConnection();
				conn.setRequestProperty("Range", "bytes=" + start + "-" + end);
				is = conn.getInputStream();
				raf = new RandomAccessFile(file, "rw");
				raf.seek(start);
				byte[] bytes = new byte[1024];
				int len;
				while ((len = is.read(bytes)) != -1) {
					raf.write(bytes, 0, len);
				}
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				try {
					raf.close();
					is.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
				checkFinish(++count);
			}
		}
		
		public void checkFinish(int count) {
			if (count == THREAD_NUM) {
				endTime = System.currentTimeMillis();
				usetime = (endTime - startTime) / 1000.0;
				System.out.println("下载用时为：" + usetime + "s");
			}
		}
	}
}
```

事不宜迟，我们赶紧把demo跑起来，先尝试一下用单线程下载网易云音乐apk文件的用时。

![usetime](/images/7/usetime1.png)

接着用3个线程来下载看看。

![usetime](/images/7/usetime2.png)

速度果然有所提升，快了大约4秒。

那么换成10个进程进行下载呢？

![usetime](/images/7/usetime3.png)

速度仍有提升，但是提升已经不大了。

-----

验证
=====
接下来我们就用多线程下载来下载一下服务器上的图片文件，看看速度是否有所提升，我们把url换成图片url。
```
private static final String PICTURE_URL = "http://101.132.185.255:8080/resource/fightpicture/gallery/QQ%E5%9B%BE%E7%89%8720180330214156.gif";
```
demo运行结果：

![usetime](/images/7/usetime4.png)

结果显示，1.8M的gif图片，仍然下载了约13秒，与之前单线程下载如出一辙。

-----

进一步探索
=====

前面通过下载网易云音乐apk文件的例子，我们知道了40M左右的文件10秒内就下载完毕了，这么来说，java性能的原因就可以直接被排除了，那么很显然，多半是服务器的速度问题。首先服务器的带宽应该不成问题，我在服务器上下载文件速度可以达到10M/S，那么多半就是tomcat的性能问题了，或许是tomcat用作静态文件服务器表现并不好。我尝试把apk文件放在了tomcat下,通过浏览器url直接去下载。

![download](/images/7/download.png)

果然，速度维持在140KB/S左右，与我们下载gif图片的速度如出一辙。

-----

结论
=====
tomcat作为文件服务器的表现性能不佳，这点我本该早点预料到。通过百度，我了解到一般通过云存储+CDN加速的方案托管我们的静态资源，例如阿里云的OSS，才能让文件下载速度真正提升上去。