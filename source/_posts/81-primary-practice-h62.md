---
url: primary-practice-h62
title: 访问控制远程文件
date: 2021-05-30 14:54:41
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h62

<!--more-->

{% note info %}

构造一个用户通过 socket 访问和控制远程文件的项目

{% endnote %}

开始理解的时候感觉很困难，但实际上就是一个文件操作的实验，套了一个 socket 的壳。

和 [h60](/primary-practice-h60) 类似，请确保你已经理解 h60。

先来看一下目录结构：

- `MyRemoteFile` 远程文件类，是远程主机下的文件
- `MyHost` 远程主机，即客户端，向服务器发出操作文件的请求
- `MyDaemon` 监听类，即服务器端，接收客户端请求，直接操作文件
- `MyDaemonConfigVo` 服务器配置
- `my_user.txt` 存放用户名和口令，`MyHost` 登录需要
- `Test` 测试文件

实际上，先配置好服务器，创建监听线程，用于响应主机的请求。然后创建远程主机，根据远程主机和路径访问并控制远程文件。

# 0x00 `MyDaemonConfigVo`

服务器配置。

在这里，服务器就是可以直接操作本地文件的一层。根据 `Test`，配置项有本地文件目录、端口号和用户信息。其中用户信息可以直接在这里读文件转换为字符串列表（不用 Map）。

```java
    private String root;
    private int port;
    private List<String> users = new ArrayList<>();
```

然后创建对应的 setter 和 getter。

读取文件的经典方法：

```java
    private List<String> readLines(String filePath) throws IOException {
        List<String> result = new ArrayList<>();
        String line;
        Reader reader = new FileReader(filePath);
        LineNumberReader lineReader = new LineNumberReader(reader);

        while (true) {
            line = lineReader.readLine();
            if (line == null) {
                break;
            }
            if (line.trim().length() == 0 || line.startsWith("#")) {
                continue;
            }
            result.add(line);
        }
        return result;
    }
```

# 0x01 `MyDaemon`

监听类，服务器端，继承 `Thread`。

创建时直接将服务器配置传过来，直接保存三个配置项：

- 本地路径，直接存字符串
- 端口号，直接创建一个 `SocketServer`
- 用户信息，直接存字符串列表

```java
    private final ServerSocket server;
    private Socket socket;
    private BufferedReader in;
    private PrintWriter out;
    private final String root;
    private final List<String> users;

    public MyDaemon(MyDeamonConfigVo config) throws IOException {
       server = new ServerSocket(config.getPort());
       socket = null;
       root = config.getRoot();
       users = config.getUsers();
       in = null;
       out = null;
    }
```

重写 `run`，循环从输入流读入并用 `readLine` 分析（进一步直接操作本地文件）：

```java
    @Override
    public void run() {
        try {
            socket = server.accept();
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream());
            while (true) {
                String line = in.readLine();
                readLine(line);
                if (1 == 0) {// 这里的条件应该是程序结束的条件，没想好怎么写
                    break;
                }
            }
            in.close();
            out.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

其中 `readLine` 是自己写的，在下文和客户端放在一起叙述。

# 0x02 `MyHost`

远程主机，客户端，需要向输出流写各种操作文件的请求，在服务器那边处理，再从输入流读入服务器输出的内容。

```java
    private String ip;
    private int port;
    private String username;
    private String password;
    private boolean valid;
    private Socket socket;
    private BufferedReader in;
    private PrintWriter out;

    public MyHost() {
        super();
        socket = null;
        in = null;
        out = null;
        valid = false;
    }
```

远程主机需要登录，即输入 `用户名\t口令` 到服务器端判断：

```java
    public void login() throws IOException {
        socket = new Socket(ip, port);
        in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        out = new PrintWriter(socket.getOutputStream());
        writeLine("login" + username + "\t" + password);
        /**
         * in.readLine() doesn't have "\r\n"
         */
        valid = "success".equals(in.readLine());
    }

    public boolean isInvalid() {
        return !valid;
    }
```

# 0x03 `MyRemoteFile` -> `MyHost` -> `MyDaemon`

`MyRemoteFile` 是远程文件类，需要一个远程主机再加上路径才能创建一个远程文件。

```java
    private final MyHost host;
    private final String path;

    public MyRemoteFile(MyHost host, String path) throws IOException {
        /**
         * 这里不能直接login，因为不只调用这个类一次，不能每次都login
         */
        if (host.isInvalid()) {// 如果主机当前是无效的，意味着主机未登录或者用户名口令不匹配
            host.login();
            if (host.isInvalid()) {
                throw new IOException("host login failed!");
            }
        }
        this.host = host;
        this.path = path;
    }
```

根据 `Test`，需要实现的功能有：

- 登录
- 列出子目录和文件
- 判断文件类型
- 获取远程文件路径（只需返回 `path`）
- 写入文件
- 删除文件
- 获取文件大小
- 检查文件是否存在

实现的过程，就是远程文件调用远程主机的方法，向监听发送请求，监听对远程主机的请求作出相应，远程主机根据响应来返回值。这里服务器端（监听）和客户端（远程主机）需要遵从一个协议，即：

| 操作类型         | 格式                                 |
| ---------------- | ------------------------------------ |
| 登录             | "login" + username + "\t" + password |
| 列出子目录和文件 | "getAscDir" + path                   |
| 判断文件类型     | "type" + path                        |
| 写入文件         | "write" + path + ":" + content       |
| 删除文件         | "delete" + path                      |
| 获取文件大小     | "length" + path                      |
| 检查文件是否存在 | "exist" + path                       |

为简化输出，定义方法 `writeLine`：

```java
private void writeLine(String line) {
    out.write(line + "\r\n");
    out.flush();
}
```

举例：为实现登录操作，客户端需要：

```java
writeLine("login" + username + "\t" + password);
```

服务器端需要：

```java
if (line.startsWith("login")) {
    checkLogin(line.substring(5));// 取"login"后面的内容
}

...

private void checkLogin(String line) {
  String result = "failed";
  for (String user: users) {
    if (line.equals(user)) {
      result = "success";
      break;
    }
  }
  writeLine(result);
}
```

如上文所叙述，服务器端始终用 `readLine` 方法处理所有请求：

```java
    private void readLine(String line) throws IOException {
        if (line.startsWith("login")) {
            checkLogin(line.substring(5));
        } else if (line.startsWith("getAscDir")) {
            listFiles(line.substring(9));
        } else if (line.startsWith("type")) {
            getFileType(line.substring(4));
        } else if (line.startsWith("write")) {
            writeFile(line.substring(5));
        } else if (line.startsWith("delete")) {
            delete(line.substring(6));
        } else if (line.startsWith("length")) {
            getLength(line.substring(6));
        } else if (line.startsWith("exist")) {
            isExist(line.substring(5));
        }
    }
```

## 登录

如上文所叙述，创建一个远程文件时需要登录主机，主机登录时依照协议向服务器端写入用户名和口令。

服务器判断请求为“登录”操作，执行 `checkLogin`，遍历用户信息字符串列表，存在则写出 `"success"` 否则 `"failed"`，客户端根据服务器的响应修改 `valid` 状态。

## 列出子目录和文件

**远程文件类：**

直接调用远程主机的方法。

```java
public MyRemoteFile[] dirByNameAsc() throws IOException, InterruptedException {
    return host.getDirByNameAsc(path);
}
```

**远程主机类：**

发出请求后，服务器先发来文件个数，据此创建远程文件数组；服务器再依次发来文件路径，据此创建每一个远程文件。

```java
public MyRemoteFile[] getDirByNameAsc(String path) throws IOException, InterruptedException {
    writeLine("getAscDir" + path);
    int count = Integer.parseInt(in.readLine());
    MyRemoteFile[] result = new MyRemoteFile[count];
    for (int i = 0; i < count; i++) {
        result[i] = new MyRemoteFile(this, in.readLine());
    }
    return result;
}
```

**监听类：**

根据请求得到文件列表，然后按照名称顺序排序，先输出文件个数再依次输出远程文件路径（远程文件根目录路径+文件名，如果是目录则再加上 `/`）。

```java
private void listFiles(String filepath) {
    List<File> files = getFiles(filepath);
    List<File> resortFiles = sortFiles(files);
    writeLine(String.valueOf(resortFiles.size()));
    for (File resortFile : resortFiles) {
        String path = filepath + resortFile.getName();
        if (!resortFile.isFile()) {
            path += "/";
        }
        writeLine(path);
    }
}

private List<File> getFiles(String filepath) {
    List<File> result = new ArrayList<>();
    File file = new File(root + filepath);
    if (!file.isFile()) {
        File[] files = file.listFiles();
        if (files != null) {
            result.addAll(Arrays.asList(files));
        }
    } else {
        result.add(file);
    }
    return result;
}

private List<File> sortFiles(List<File> fileList) {
    List<File> result = new ArrayList<>();
    List<File> files = new ArrayList<>();
    List<File> dirs = new ArrayList<>();

    for (File file: fileList) {
        if (file.isFile()) {
            files.add(file);
        } else {
            dirs.add(file);
        }
    }
    /**
     * it will be changed
     */
    int dirCount = dirs.size();
    int fileCount = files.size();

    for (int i = 0; i < dirCount; i++) {
        File dir = dirs.get(0);
        for (File file: dirs) {
            if (dir.getAbsolutePath().compareTo(file.getAbsolutePath()) > 0) {
                dir = file;
            }
        }
        result.add(dir);
        dirs.remove(dir);
    }
    for (int i = 0; i < fileCount; i++) {
        File file = files.get(0);
        for (File file1: files) {
            if (file.getAbsolutePath().compareTo(file1.getAbsolutePath()) >= 0) {
                file = file1;
            }
        }
        result.add(file);
        files.remove(file);
    }
    return result;
}
```

## 判断文件类型

**远程文件类：**

规定一个文件如果是目录则返回 0，如果是文件则返回 1，否则返回 -1。

```java
public boolean isDirectory() throws IOException {
    return host.getType(path) == 0;
}

public boolean isFile() throws IOException {
    return host.getType(path) == 1;
}
```

**远程主机类：**

规定一个文件如果是目录则输出 `"dir"`，如果是文件则输出 `"file"`。

```java
public int getType(String path) throws IOException {
    writeLine("type" + path);
    String type = in.readLine();
    if ("file".equals(type)) {
        return 1;
    } else if ("dir".equals(type)) {
        return 0;
    } else {
        return -1;
    }
}
```

**监听类：**

根据路径创建本地文件对象，判断文件类型。

```java
private void getFileType(String filepath) {
    File file = new File(root + filepath);
    if (file.isFile()) {
        writeLine("file");
    } else {
        writeLine("dir");
    }
}
```

## 获取远程文件路径

**远程文件类：**

```java
public String getPathFileName() {
    return path;
}
```

## 写入文件

**远程文件类：**

直接调用远程主机，传入路径和文件内容。

```java
public void writeByBytes(byte[] bytes) {
    host.writeByBytes(path, bytes);
}
```

**远程主机类：**

根据字节数组创建字符串。

```java
public void writeByBytes(String path, byte[] bytes) {
    String content = new String(bytes, StandardCharsets.UTF_8);
    writeLine("write" + path + ":" + content);
}
```

**监听类：**

根据格式分离路径和文件内容，文件不存在则新建文件，然后通过文件输出流输出文件内容。

```java
private void writeFile(String pathAndContent) throws IOException {
    String[] pathContent = pathAndContent.split(":");
    String path = pathContent[0];
    String content = pathContent[1];
    File file = new File(root + path);
    if (!file.exists()) {
        if (file.createNewFile()) {
            FileOutputStream outFileStream = new FileOutputStream(file);
            outFileStream.write(content.getBytes(StandardCharsets.UTF_8));
            outFileStream.flush();
        }
    }
}
```

## 删除文件

**远程文件类：**

直接调用远程主机。

```java
public void delete() {
    host.delete(path);
}
```

**远程主机类：**

按照格式发送请求。

```java
public void delete(String path) {
    writeLine("delete" + path);
}
```

**监听类：**

由于客户端没有读取操作，这里也不能输出。

{% note warning %}

`file.delete()` 是有返回值的。

{% endnote %}

```java
private void delete(String filepath) {
    File file = new File(root + filepath);
    /**
     * write must be read
     */
    if (file.delete()) {
    }
}
```

## 获取文件大小

**远程文件类：**

直接调用远程主机。

```java
public int length() throws IOException {
    return host.getLength(path);
}
```

**远程主机类：**

按照格式发送请求，将返回的字符串转换为数字。

```java
public int getLength(String path) throws IOException {
    writeLine("length" + path);
    String len = in.readLine();
    return Integer.parseInt(len);
}
```

**监听类：**

根据路径新建文件，若存在则输出文件大小（转换为字符串），否则输出 `"0"`。

```java
private void getLength(String filepath) {
    File file = new File(root + filepath);
    if (file.exists()) {
        writeLine(String.valueOf(file.length()));
    } else {
        writeLine("0");
    }
}
```

## 检查文件是否存在

**远程文件类：**

直接调用远程主机。

```java
public boolean exists() throws IOException {
    return host.isExist(path);
}
```

**远程主机类：**

按照格式发送请求，对服务器输出的结果进行判断。

```java
public boolean isExist(String path) throws IOException {
    writeLine("exist" + path);
    String result = in.readLine();
    return "exist".equals(result);
}
```

**监听类：**

新建文件，判断是否在本地存在。

```java
private void isExist(String filepath) {
    File file = new File(root + filepath);
    if (file.exists()) {
        writeLine("exist");
    } else {
        writeLine("not exist");
    }
}
```
