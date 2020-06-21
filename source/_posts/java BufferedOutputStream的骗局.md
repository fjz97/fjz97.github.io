---
title: java BufferedOutputStream的骗局
date: 2018-04-04 19:13:14
tags:
- java
categories:
- 技术
---
文章的开始，先从一个案例谈起。
最近在做的项目中有图片保存的需求，之前一律按jpg格式处理了，今天想想不妥，于是适配了所有的图片格式。代码如下：

```
        try {
            FileOutputStream fos = new FileOutputStream(file);
            int length;
            byte[] bytes = new byte[10240];
            while((length = is.read(bytes, 0, 10240)) != -1) {
                fos.write(bytes, 0, length);
            }
            fos.close();
            is.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
```

项目跑起来，试着保存了一张gif图片，显示成功，本以为大功告成，结果切到本地图片gallery一看，图片根本没有保存成功。正愣着呢，图片又突然出现了。我稍微一想，就知道是保存图片的速度过慢。输出看了下缓存图片的耗时。

![usetime](/images/6/usetime.png)

2MB的gif图片足足缓存了十几秒，用户体验想必是很差的，那么有没有办法加快缓存速度呢？我的java知识告诉我，有！使用BufferedOutputStream维护一个字节缓冲池以减少IO次数，提升缓存速度。于是我使用了BufferedOutputStream：

```
            FileOutputStream fos = new FileOutputStream(file);
            BufferedOutputStream bos = new BufferedOutputStream(fos);
            int length;
            byte[] bytes = new byte[10240];
            while((length = is.read(bytes, 0, 10240)) != -1) {
                bos.write(bytes, 0, length);
            }
            bos.flush();
            bos.close();
            is.close();
```

这回跑起来肯定快了吧！结果：

![usetime2](/images/6/usetime2.png)

根本没有变快嘛！怎么回事？难道我记错了？抱着疑惑，我打开了BufferedOutputStream源码：

```
public
class BufferedOutputStream extends FilterOutputStream {
    /**
     * The internal buffer where data is stored.
     */
    protected byte buf[];

    /**
     * The number of valid bytes in the buffer. This value is always
     * in the range <tt>0</tt> through <tt>buf.length</tt>; elements
     * <tt>buf[0]</tt> through <tt>buf[count-1]</tt> contain valid
     * byte data.
     */
    protected int count;

    /**
     * Creates a new buffered output stream to write data to the
     * specified underlying output stream.
     *
     * @param   out   the underlying output stream.
     */
    public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }

    /**
     * Creates a new buffered output stream to write data to the
     * specified underlying output stream with the specified buffer
     * size.
     *
     * @param   out    the underlying output stream.
     * @param   size   the buffer size.
     * @exception IllegalArgumentException if size &lt;= 0.
     */
    public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }

    /** Flush the internal buffer */
    private void flushBuffer() throws IOException {
        if (count > 0) {
            out.write(buf, 0, count);
            count = 0;
        }
    }

    /**
     * Writes the specified byte to this buffered output stream.
     *
     * @param      b   the byte to be written.
     * @exception  IOException  if an I/O error occurs.
     */
    public synchronized void write(int b) throws IOException {
        if (count >= buf.length) {
            flushBuffer();
        }
        buf[count++] = (byte)b;
    }

    /**
     * Writes <code>len</code> bytes from the specified byte array
     * starting at offset <code>off</code> to this buffered output stream.
     *
     * <p> Ordinarily this method stores bytes from the given array into this
     * stream's buffer, flushing the buffer to the underlying output stream as
     * needed.  If the requested length is at least as large as this stream's
     * buffer, however, then this method will flush the buffer and write the
     * bytes directly to the underlying output stream.  Thus redundant
     * <code>BufferedOutputStream</code>s will not copy data unnecessarily.
     *
     * @param      b     the data.
     * @param      off   the start offset in the data.
     * @param      len   the number of bytes to write.
     * @exception  IOException  if an I/O error occurs.
     */
    public synchronized void write(byte b[], int off, int len) throws IOException {
        if (len >= buf.length) {
            /* If the request length exceeds the size of the output buffer,
               flush the output buffer and then write the data directly.
               In this way buffered streams will cascade harmlessly. */
            flushBuffer();
            out.write(b, off, len);
            return;
        }
        if (len > buf.length - count) {
            flushBuffer();
        }
        System.arraycopy(b, off, buf, count, len);
        count += len;
    }

    /**
     * Flushes this buffered output stream. This forces any buffered
     * output bytes to be written out to the underlying output stream.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FilterOutputStream#out
     */
    public synchronized void flush() throws IOException {
        flushBuffer();
        out.flush();
    }
}
```

我没记错，BufferedOutputStream确实维护了一个byte数组，先往byte数组里面填充字节，当byte数组满了，再写出去。但是，如果不设置参数，缓冲池的大小只有8192byte，也就是说，我10240byte一次的写出大小反而增多了IO次数，也难怪速度没有提升。我试着加大了BufferedOutputStream的buffer大小：

`BufferedOutputStream bos = new BufferedOutputStream(fos, 1000 * 1024);`

结果：

![usetime3](/images/6/usetime3.png)

速度还是没有提升，那么只有一种可能了，IO根本不是影响缓存速度的决定因素。或者说，这么点次数的IO不会是影响缓存速度的决定因素。为了证实我的猜想，我们试试看每次只读入写出1个字节，增多读入写出次数，增大IO开销：

```
            FileOutputStream fos = new FileOutputStream(file);
            int data;
            while((data = is.read()) != -1) {
                fos.write(data);
            }
            fos.close();
            is.close();
```

结果：

![usetime4](/images/6/usetime4.png)

终于，在这种极端情况下，缓存图片的时间终于增长了！本来是费尽心机想减少缓存时间，现在竟然费尽心机想增长缓存时间！

既然IO不是影响缓存速度的决定因素，那么，使用BufferedOutputStream的意义似乎就不存在了。

**另外，如果想要减少IO次数，直接增大代码中的byte数组不就可以了吗，别忘了，我们使用read(bytes[], int, int)和write(bytes[], int, int)这种有别于无参数的read和一个参数的write方法，其实就是做了和BufferedOutputStream一样的工作——维护一个缓冲池。这么一看，BufferedOutputStream的存在意义，似乎也不存在了。**