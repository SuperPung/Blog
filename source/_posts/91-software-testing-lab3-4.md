---
url: software-testing-lab3-4
title: 软件测试技术实验 3 和 4 遇到的一些问题和解决方案
date: 2022-03-23 22:02:56
categories: [技术]
tags: [Java, 软件测试]
---

最近写《软件测试技术》实验 3 和实验 4 时，遇到了一些工具上的问题，并尝试了一些解决方案，记录一下。

<!--more-->

# 实验 3：MuJava

[MuJava](https://cs.gmu.edu/~offutt/mujava/) 是一个变异测试的工具，可以生成变异体并运行，最终可以得到变异测试的结果。

> 变异测试是衡量测试充分性的有效方法，其思想就是模拟程序编写中可能会出现的错误，得到一系列和源程序不同的“变异体”，用来“测试”你的测试程序是否能测出来这些“变异”。你的测试程序杀死（使变异体的运行结果不同于源程序）的变异体越多，测试越充分。

## 0x00 安装 MuJava

1. 直接到 [官网](https://cs.gmu.edu/~offutt/mujava/) 下载 `mujava.jar`。

2. 为了后续实验进行，还需要下载 `junit.jar`（编写测试程序）、`openjava.jar`。

3. 保存三个 jar 包到固定路径，编辑 `~/.bash_profile` 引入环境：

	```shell
	export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home
	export PATH=$PATH:$JAVA_HOME/bin
	export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:/path/to/openjava.jar:/path/to/junit.jar:/path/to/mujava.jar
	```

	（适用于 macOS，Windows 略有不同）

4. 保存并运行 `source ~/.bash_profile`。

5. 为了使用 MuJava，需要创建需要的目录结构：

	```
	|--\classes\
	|--mujava.config
	|--\result\
	|--\src\
	|--\testset\
	```

## 0x01 生成变异体

1. 将源文件移至 `\src\` 目录。
2. `javac` 编译源文件，得到 `classes` 文件，移至 `\classes\` 目录。
3. 运行 `java mujava.gui.GenMutantsMain`，进入图形化设置界面，勾选需要生成变异体的文件和需要的变异算子，点击 `Generate` 即可生成（保存在 `result` 目录）。

{% note warning %}

若此处出现问题，请检查 0x00 第 3 步中路径是否包含空格。即使使用 `\` 转译也可能存在问题，建议用 `-` 或 `_` 代替空格。

{% endnote %}

## 0x02 生成测试集

使用 JUnit 编写即可，比较简单，得到 `xxxTest.java`。

## 0x03 运行变异测试

1. `javac` 编译测试文件，得到 `classes` 文件，移至 `\testset\` 目录。
2. 运行 `java mujava.gui.RunTestMain > TestResult.log`，将变异测试日志输出至文件保存。

{% note warning %}

若此处出现问题，请检查 0x00 第 3 步中是否引入了 `$JAVA_HOME/lib/tools.jar` 和 `$JAVA_HOME/lib/dt.jar`。

{% endnote %}

变异测试完成，后续可以进一步分析测试代码和变异体，做进一步改进。

# 实验 4：Major

[Major](http://mutation-testing.org/) 和 Mujava 功能相近，也是变异测试的工具。

## 0x00 安装 Major

1. 直接到 [官网](http://mutation-testing.org/downloads/) 下载 Major v1.3.5，解压为 major 目录，存放到固定路径。

2. 配置环境变量：

	```shell
	export PATH=/path/to/major/bin:$PATH:$JAVA_HOME/bin
	```

	保存并运行 `source ~/.bash_profile`。

3. 验证是否配置成功：

	```shell
	❯ javac -version
	javac 1.7.0-Major-v1.3.5
	❯ ant -version
	Apache Ant(TM) version 1.8.4-Major-v1.3.5 compiled on July 18 2019
	```

{% note warning %}

如果验证时得到的输出和上述不同，特别是 `javac -version` 输出结果不包含 `Major-v1.3.5`，建议检查第 2 步引入环境变量的顺序，应将 `major/bin` 目录放在最前面。

{% endnote %}

## 0x01 配置文件

1. Major 使用 `ant` 运行变异测试，并在 `major/example/` 目录下给出运行模板。所以我们需要依此建立目录结构：

	```
	|--\src\
	|--\test\
	|--build.xml
	|--run.sh
	```

2. 将源文件移至 `src` 目录，将 `JUnit` 测试文件移至 `test` 目录。

{% note warning %}

请确保代码文件的 `package` 和目录结构对应，参照给出的运行模板。

{% endnote %}

`build.xml` 和 `run.sh` 是 Major 运行的重要文件，前者是配置文件、后者是运行脚本。由于运行目录不同，所以在运行前我们需要在 `example` 的基础上，根据自己的实际情况对两个文件做一些修改。

### `build.xml`

1. 可以在第一行修改自己的项目名称：

	```xml
	<project name="YourProjectName" default="compile" basedir=".">
	```

2. 修改 `/javac/` 文件的路径：

	```xml
	<property name="major" value="/your/path/to/major/bin/javac"/>
	```

	它上面两行一般不需要改动。

### `run.sh`

1. 修改 `/major/` 目录的路径：

	```shell
	MAJOR_HOME="/your/path/to/major"
	```

2. 如果没有编写 MML 脚本（一般情况下不需要自己编写），修改 `ant` 运行的参数：

	```shell
	$MAJOR_HOME/bin/ant -DmutOp=":ALL" clean compile
	```

	直接指定为 `ALL` 对应全部变异算子，若此处不修改可能会导致后续生成变异体数量为 0。

## 0x02 运行变异测试

运行 `./run.sh`，得到类似下方的输出则成功：

```shell
...
mutation.test:
     [echo] Running mutation analysis ...
    [junit] MAJOR: Mutation analysis enabled
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Run 1 ordered test to verify independence
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Preprocessing time: xxx seconds
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Mutants generated: xxx
    [junit] MAJOR: Mutants covered:   xxx (xx.xx%)
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Export test map to testMap.csv
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Run mutation analysis with 1 individual test
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: 1/1 - UpgradedTriangleTest (xxms / xxx):
    [junit] MAJOR: xxx (xxx / xxx / xxx) -> AVG-RTPM: xms
    [junit] MAJOR: Mutants killed / live: xxx (xxx-x-x) / xx
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Summary:
    [junit] MAJOR:
    [junit] MAJOR: Analysis time:  x.x seconds
    [junit] MAJOR: Mutation score: xx.xx% (xx.xx%)
    [junit] MAJOR: Mutants killed / live: xxx (xxx-x-x) / xx
    [junit] MAJOR: Mutant executions: xxx
    [junit] MAJOR: ------------------------------------------------------------
    [junit] MAJOR: Export summary of results to summary.csv
    [junit] MAJOR: Export run-time results to results.csv
    [junit] MAJOR: Export mutant kill details to killed.csv

BUILD SUCCESSFUL
Total time: x second
```

和 MuJava 类似，后续可以进一步分析测试代码和变异体，做进一步改进。

{% note warning %}

如果 Mutants covered 未达到 100%，请移除源程序的 `main` 方法。

{% endnote %}

## 0x03 其他问题

如有其他问题，强烈建议阅读 [官方文档](http://mutation-testing.org/doc/major.pdf)，十分详细。
