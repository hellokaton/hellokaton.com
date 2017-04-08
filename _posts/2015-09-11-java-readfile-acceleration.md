---
layout: post
title: Java读取文件加速
categories: java
tags: io
---

在执行IO时，Java的InputStream被广泛使用，比如DataInputStream.readInt等等。事实上，这些高度封装的接口奇慢无比。我有一个项目启动时需要读取90MB左右的词典文件，用DataInputStream耗时3秒以上，换用java.nio包直接操作内存字节，可以加速到300ms左右，整整提速10倍！当然，前提是你熟悉位运算。

<!-- more -->

java.nio中提供了两个类 `FileChannel ` 和 `ByteBuffer` 来将文件映射到内存，其中`FileChannel`表示文件通道，`ByteBuffer`是一个缓冲区。

## 具体步骤

1. 从`FileInputStream`、`FileOutputStream`以及`RandomAccessFile`对象获取文件通道
2. 将文件内存映射到`ByteBuffer`
3. 通过`byteBuffer.array()`接口得到一个`byte`数组
4. 直接操作字节

**示例代码**

```java
FileInputStream fis = new FileInputStream(path);
// 1.从FileInputStream对象获取文件通道FileChannel
FileChannel channel = fis.getChannel();
int fileSize = (int) channel.size();

// 2.从通道读取文件内容
ByteBuffer byteBuffer = ByteBuffer.allocate(fileSize);

// channel.read(ByteBuffer) 方法就类似于 inputstream.read(byte)
// 每次read都将读取 allocate 个字节到ByteBuffer
channel.read(byteBuffer);
// 注意先调用flip方法反转Buffer,再从Buffer读取数据
byteBuffer.flip();
// 可以将当前Buffer包含的字节数组全部读取出来
byte[] bytes = byteBuffer.array();
byteBuffer.clear();
// 关闭通道和文件流
channel.close();
fis.close();

int index = 0;
size = Utility.bytesHighFirstToInt(bytes, index);
index += 4;
```

其中，如果你当初使用了DataOutputStream.writeInt来保存文件的话，那么在读取的时候就要注意了。writeInt写入四个字节，其中高位在前，低位在后，所以将byte数组转为int的时候需要倒过来转换：

```java
 /**
 * 字节数组和整型的转换，高位在前，适用于读取writeInt的数据
 *
 * @param bytes 字节数组
 * @return 整型
 */
public static int bytesHighFirstToInt(byte[] bytes, int start) {
    int num = bytes[start + 3] & 0xFF;
    num |= ((bytes[start + 2] << 8) & 0xFF00);
    num |= ((bytes[start + 1] << 16) & 0xFF0000);
    num |= ((bytes[start] << 24) & 0xFF000000);
    return num;
}
```