---
layout: post
title: java复制文件的4种方式
categories: java
tags: io
---

尽管Java提供了一个可以处理文件的IO操作类。
但是没有一个复制文件的方法。
复制文件是一个重要的操作,当你的程序必须处理很多文件相关的时候。
然而有几种方法可以进行Java文件复制操作,下面列举出4中最受欢迎的方式。

<!-- more -->

### 1. 使用FileStreams复制

这是最经典的方式将一个文件的内容复制到另一个文件中。
使用FileInputStream读取文件A的字节，使用FileOutputStream写入到文件B。
这是第一个方法的代码:

```java
private static void copyFileUsingFileStreams(File source, File dest)
		throws IOException {
	InputStream input = null;
	OutputStream output = null;
	try {
		input = new FileInputStream(source);
		output = new FileOutputStream(dest);
		byte[] buf = new byte[1024];
		int bytesRead;
		while ((bytesRead = input.read(buf)) > 0) {
			output.write(buf, 0, bytesRead);
		}
	} finally {
		input.close();
		output.close();
	}
}
```

正如你所看到的我们执行几个读和写操作try的数据,所以这应该是一个低效率的,下一个方法我们将看到新的方式。

### 2. 使用FileChannel复制

Java NIO包括transferFrom方法,根据文档应该比文件流复制的速度更快。
这是第二种方法的代码:

```java
private static void copyFileUsingFileChannels(File source, File dest)
		throws IOException {
	FileChannel inputChannel = null;
	FileChannel outputChannel = null;
	try {
		inputChannel = new FileInputStream(source).getChannel();
		outputChannel = new FileOutputStream(dest).getChannel();
		outputChannel.transferFrom(inputChannel, 0, inputChannel.size());
	} finally {
		inputChannel.close();
		outputChannel.close();
	}
}
```

### 3. 使用Commons IO复制

Apache Commons IO提供拷贝文件方法在其FileUtils类,可用于复制一个文件到另一个地方。它非常方便使用Apache Commons FileUtils类时,您已经使用您的项目。基本上,这个类使用Java NIO FileChannel内部。

这是第三种方法的代码:

```bash
private static void copyFileUsingApacheCommonsIO(File source, File dest)
		throws IOException {
	FileUtils.copyFile(source, dest);
}
```

### 4. 使用Java7的Files类复制

如果你有一些经验在Java 7中你可能会知道,可以使用复制方法的Files类文件,从一个文件复制到另一个文件。

这是第四个方法的代码:

```bash
private static void copyFileUsingJava7Files(File source, File dest)
		throws IOException {
	Files.copy(source.toPath(), dest.toPath());
}
```

### 测试

现在看到这些方法中的哪一个是更高效的,我们会复制一个大文件使用每一个在一个简单的程序。
从缓存来避免任何性能明显我们将使用四个不同的源文件和四种不同的目标文件。

让我们看一下代码:

```java

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.nio.channels.FileChannel;
import java.nio.file.Files;
import org.apache.commons.io.FileUtils;

public class CopyFilesExample {

	public static void main(String[] args) throws InterruptedException,
			IOException {

		File source = new File("C:\\Users\\nikos7\\Desktop\\files\\sourcefile1.txt");
		File dest = new File("C:\\Users\\nikos7\\Desktop\\files\\destfile1.txt");

		// copy file using FileStreams
		long start = System.nanoTime();
		long end;
		copyFileUsingFileStreams(source, dest);
		System.out.println("Time taken by FileStreams Copy = "
				+ (System.nanoTime() - start));

		// copy files using java.nio.FileChannel
		source = new File("C:\\Users\\nikos7\\Desktop\\files\\sourcefile2.txt");
		dest = new File("C:\\Users\\nikos7\\Desktop\\files\\destfile2.txt");
		start = System.nanoTime();
		copyFileUsingFileChannels(source, dest);
		end = System.nanoTime();
		System.out.println("Time taken by FileChannels Copy = " + (end - start));

		// copy file using Java 7 Files class
		source = new File("C:\\Users\\nikos7\\Desktop\\files\\sourcefile3.txt");
		dest = new File("C:\\Users\\nikos7\\Desktop\\files\\destfile3.txt");
		start = System.nanoTime();
		copyFileUsingJava7Files(source, dest);
		end = System.nanoTime();
		System.out.println("Time taken by Java7 Files Copy = " + (end - start));

		// copy files using apache commons io
		source = new File("C:\\Users\\nikos7\\Desktop\\files\\sourcefile4.txt");
		dest = new File("C:\\Users\\nikos7\\Desktop\\files\\destfile4.txt");
		start = System.nanoTime();
		copyFileUsingApacheCommonsIO(source, dest);
		end = System.nanoTime();
		System.out.println("Time taken by Apache Commons IO Copy = "
				+ (end - start));

	}

	private static void copyFileUsingFileStreams(File source, File dest)
			throws IOException {
		InputStream input = null;
		OutputStream output = null;
		try {
			input = new FileInputStream(source);
			output = new FileOutputStream(dest);
			byte[] buf = new byte[1024];
			int bytesRead;
			while ((bytesRead = input.read(buf)) > 0) {
				output.write(buf, 0, bytesRead);
			}
		} finally {
			input.close();
			output.close();
		}
	}

	private static void copyFileUsingFileChannels(File source, File dest)
			throws IOException {
		FileChannel inputChannel = null;
		FileChannel outputChannel = null;
		try {
			inputChannel = new FileInputStream(source).getChannel();
			outputChannel = new FileOutputStream(dest).getChannel();
			outputChannel.transferFrom(inputChannel, 0, inputChannel.size());
		} finally {
			inputChannel.close();
			outputChannel.close();
		}
	}

	private static void copyFileUsingJava7Files(File source, File dest)
			throws IOException {
		Files.copy(source.toPath(), dest.toPath());
	}

	private static void copyFileUsingApacheCommonsIO(File source, File dest)
			throws IOException {
		FileUtils.copyFile(source, dest);
	}

}
```

#### 输出：

```bash
Time taken by FileStreams Copy = 127572360
Time taken by FileChannels Copy = 10449963
Time taken by Java7 Files Copy = 10808333
Time taken by Apache Commons IO Copy = 17971677
```

正如您可以看到的FileChannels拷贝大文件是最好的方法。如果你处理更大的文件,你会注意到一个更大的速度差。
这是一个示例,该示例演示了Java中四种不同的方法可以复制一个文件。