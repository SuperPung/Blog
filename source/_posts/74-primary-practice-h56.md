---
url: primary-practice-h56
title: 文本文件内容替换
date: 2021-05-06 21:12:02
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h56

<!--more-->

{% note info %}

将 homeDir 目录下（包括子目录）所有的文本文件（扩展名为 .txt，扩展名不是 .txt 的文件不要动，扩展名区分大小写) 文件中，orgStr 替换为 targetStr

所有文本文件均为 UTF-8 编码

{% endnote %}

与简单的字符串 replace 不同，此处涉及文件的写入操作。

为方便替换，在 FileTool 中定义属性：

```java
	private static final String EXTNAME = "txt";
	private String orgStr = "";
	private String targetStr = "";
```

根据题目要求，需要递归遍历文件，所以在 `replaceTxtFileContent` 方法中给属性赋值后再调用 `readFiles` 方法：

```java
	public void replaceTxtFileContent(String homeDir,String orgStr,String targetStr) {
		this.orgStr = orgStr;
		this.targetStr = targetStr;
		File home = new File(homeDir);
		readFiles(home);
	}
```

在经典的递归读文件方法中调用替换文件内容方法：

```java
	private void readFiles(File dir) {
		if (!dir.exists() || !dir.isDirectory()) {
			return;
		}
		String[] files = dir.list();
		if (files == null) {
			return;
		}
		for (String s : files) {
			File file = new File(dir, s);
			if (file.isFile()) {
				String filename = file.getName();
				if (isTxtFile(filename)) {
					replaceFileContent(file);
				}
			} else {
				readFiles(file);
			}
		}
	}
```

在替换文件内容方法中，用最简单理解的方式，先读出文件内容到字符串中，再替换其中的字符串，再将替换后的字符串写入文件：

```java
	private void replaceFileContent(File file) {
		String content = readFromTxt(file);
		if (content.length() == 0) {
			return;
		}
		String newContent = content.replace(orgStr, targetStr);
		try {
			FileWriter fw = new FileWriter(file.getAbsoluteFile());
			BufferedWriter bw = new BufferedWriter(fw);
			bw.write(newContent);
			bw.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

替换之前需要判断是否为文本文件，直接取扩展名再无大小判断即可：

```java
	private boolean isTxtFile(String filename) {
		String[] filenames = filename.split("\\.");
		String ext = filenames[filenames.length - 1];
		return EXTNAME.equalsIgnoreCase(ext);
	}
```

经典读取文本文件的方法：

```java
	private String readFromTxt(File file) {
		Reader reader = null;
		StringBuffer buf = new StringBuffer();
		try {
			char[] chars = new char[1024];
			// InputStream in=new FileInputStream(filename);
			reader = new InputStreamReader(new FileInputStream(file), StandardCharsets.UTF_8);
			int readed = reader.read(chars);
			while (readed != -1) {
				buf.append(chars, 0, readed);
				readed = reader.read(chars);
			}
		} catch(Exception e) {
			e.printStackTrace();
		} finally {
			close(reader);
		}
		return buf.toString();
	}

	private void close(Closeable inout) {
		if (inout != null) {
			try {
				inout.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
```
