---
url: primary-practice-h60
title: 简单的聊天室
date: 2021-05-22 16:51:20
categories: [技术]
tags: [程序设计实践]
---

Primary Practice h60

<!--more-->

{% note info %}

在 `ChatServer`、`ChatClient` 中增加适当代码，并增加适当的类，完成一个简单的聊天室

{% endnote %}

和之前 OOP 的一个实验类似，利用 Socket 实现 client 和 server 通信。不同的是，本实验需要多客户端和一服务器通信。

# 0x00 ChatServer

类变量一定包括一个 `ServerSocket`。

根据要求，需要从文件中读取到所有的用户名和密码，用 `HashMap` 保存。

多 client，为了给每一个 client 回复，需要保存目前已经连接的 client。

```java
	private final ServerSocket server;
	private final Map<String, String> passwd;
	public static List<Socket> clients = new ArrayList<>();
```

构造方法只需要初始化 server 和 passwd：

```java
	public ChatServer (int port, String passwordFilename) throws IOException {
		server = new ServerSocket(port);
		passwd = readLines(passwordFilename);
	}
```

经典：

```java
	public Map<String, String> readLines(String filename) throws IOException {
		String line;
		Reader reader;
		Map<String, String> result = new HashMap<>();
		reader = new FileReader(filename);
		LineNumberReader lineReader = new LineNumberReader(reader);

		while (true) {
			line = lineReader.readLine();
			if (line == null) {
				break;
			}
			if (line.trim().length() == 0 || line.startsWith("#")) {
				continue;
			}
			String[] lineContent = line.split("\t");
			result.put(lineContent[0], lineContent[1]);
		}
		return result;
	}
```

需要开始监听，所以此类一定继承了 `Thread`，`startListen` 只需要调用 `run` 方法：

```java
	public void startListen() {
		this.start();
	}
```

重写 `run`，因为要持续监听，与多个 client 相连，所以需要新建 `ServerThread` 类，在此类中调用它：

```java
	@Override
	public void run() {
		while (true) {
			Socket socket = null;
			try {
				socket = server.accept();
			} catch (IOException e) {
				e.printStackTrace();
			}
			clients.add(socket);
			new ServerThread(socket, passwd).start();
		}
	}
```

# 0x01 ServerThread

构造方法：

```java
    private final Socket socket;
    private final Map<String, String> passwd;

    public ServerThread(Socket socket, Map<String, String> passwd) {
        super();
        this.socket = socket;
        this.passwd = passwd;
    }
```

重写 `run`，从输入流读入，向输出流写出。读入的是每一个客户端说的话，写出需要向所有客户端写出。

{% note warning %}

这里不知道怎么处理比较合适，可能会有更好的写法，欢迎提 pr。

{% endnote %}

```java
    @Override
    public void run() {
        try {
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream());
            String line = in.readLine();
            String passwdStr = line;
//            System.out.println("server received username and password: " + passwdStr);
            String isLoggedIn = pass(passwdStr);
            out.write(isLoggedIn + "\r\n");
            out.flush();
            if ("0".equals(isLoggedIn)) {
                return;
            }
            while (true) {
                line = in.readLine();
                // \r\n is necessary
                for (Socket client : ChatServer.clients) {
                    PrintWriter clientOut = new PrintWriter(client.getOutputStream());
                    clientOut.write(line + "\r\n");
                    clientOut.flush();
                }
                if (1 == 0) {
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

判断用户名和口令是否有效，为了方便就直接返回字符串了：

```java
    private String pass(String passwdStr) {
        String[] userPasswd = passwdStr.split("\t");
        if (userPasswd.length != 2) {
            return "0";
        }
        String username = userPasswd[0];
        String password = userPasswd[1];
        Set<Map.Entry<String, String>> passwdEntry = passwd.entrySet();
        for (Map.Entry<String, String> entry : passwdEntry) {
            if (username.equals(entry.getKey())) {
                if (password.equals(entry.getValue())) {
                    return "1";
                } else {
                    return "0";
                }
            }
        }
        return "0";
    }
```

# 0x03 ChatClient

每个客户端有是否登录两种状态：

```java
	private boolean isLoggedIn;
	private final PrintWriter out;
	private final BufferedReader in;
```

构造方法：

```java
	public ChatClient (String ip, int port) throws IOException {
		Socket client = new Socket(ip, port);
		// 获得输出流
		out = new java.io.PrintWriter(client.getOutputStream());
		// 获得输入流
		in = new BufferedReader(new InputStreamReader(client.getInputStream()));
	}
```

登录操作，向服务器写入用户名和密码（用 `\t` 分隔），服务器会判断并写出到这里的输入流。

是不是有更好的写法？欢迎提 pr。

```java
	public boolean login(String userName,String password) throws IOException {
		String passwd = userName + "\t" + password;
		String get;
		// \r\n is necessary
		out.write(passwd + "\r\n");
		out.flush();
		get = in.readLine();
		isLoggedIn = "1".equals(get);
		return isLoggedIn;
	}
```

登出：

```java
	public void logout() {
		isLoggedIn = false;
	}
```

发言：

```java
	public void speak(String str) throws IOException {
		 if (!isLoggedIn) {
		 	throw new IOException("Haven't logged in!");
		 }
		 out.write(str + "\r\n");
		 out.flush();
	}
```

读取发言：

这里是否需要创建消息队列？欢迎评论、issue、提 pr 交流。

```java
	public String read() throws IOException {
		if (!isLoggedIn) {
			return null;
		}
		return in.readLine();
	}
```

---

有很多地方写的不是很好，但最后也过了，可能需要特定的测试用例？
