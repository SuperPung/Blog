---
url: vscode-config
title: VS Code 配置
date: 2021-01-29 11:08:53
categories: [技术]
tags: [VS Code]
---

Windows 安装并配置 Visual Studio Code 过程记录

<!--more-->

> ![Visual Studio Code 1.35 icon.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9a/Visual_Studio_Code_1.35_icon.svg/64px-Visual_Studio_Code_1.35_icon.svg.png)
>
> Visual Studio Code（简称 VS Code）是一个由微软开发，同时支持 Windows、Linux 和 macOS 等操作系统的免费代码编辑器，它支持测试，并内置了 Git 版本控制功能，同时也具有开发环境功能，例如代码补全（类似于 IntelliSense）、代码片段和代码重构等。该编辑器支持用户个性化配置，例如改变主题颜色、键盘快捷方式等各种属性和参数，同时还在编辑器中内置了扩展程序管理的功能。
>
> 在 2019 年的 Stack Overflow 组织的开发者调研中，VS Code 被认为是最受开发者欢迎的开发环境，据调查 87317 名受访者中有 50.7% 的受访者声称正在使用VS Code。
>
> ---
>
> *参考文献*
>
> *[1]维基百科编者. Visual Studio Code[G/OL]. 维基百科, 2020(20201221)[2020-12-21]. https://zh.wikipedia.org/w/index.php?title=Visual_Studio_Code&oldid=63349035.*

本文使用的软件版本及系统环境：

- Windows 10
- Visual Studio Code x64 1.52.1
- MinGW-w64 GCC-8.1.0

本文部分代码参考[知乎@谭九鼎](https://www.zhihu.com/question/30315894/answer/154979413)。

更多详细内容请参考[VS Code 官方文档](https://code.visualstudio.com/docs)。

# 下载 VS Code

访问[VS Code 官网](https://code.visualstudio.com/)，点击左侧“Download for Windows”即可下载。

> 下载速度慢？试试将下载地址中的“az764295.vo.msecnd.net”替换为国内镜像“vscode.cdn.azure.cn”。

# 安装 VS Code

双击打开下载好的 exe 文件，选择“我同意此协议”，选择目标位置，选择开始菜单文件夹。

选择附加任务，勾选“将‘通过 Code 打开’操作添加到 Windows 资源管理器文件、目录上下文菜单”两项：

![code安装1](https://i0.hdslb.com/bfs/album/ddd83795f430c520e20d91adb79ab0855ce71451.jpg)

下一步，点击“安装”，稍等即可安装成功。

# 下载编译器 MinGW-w64

VS Code 只是一个文本编辑器（text editor），而不是 IDE（集成开发环境）。所以要想编译并运行程序，需要安装编译器。

> Mingw-w64 是自由及开放源代码软件开发环境，用于创建 Microsoft Windows 应用程序。从 2005–2008 从 MinGW (*Minimalist GNU for Windows*)分枝出来。
>
> Mingw-w64 包括对 GCC、GNU Binutils 的 Windows 版本的移植（汇编器、链接器、库文件管理器），一套自由可分发的 Windows 特定的头文件与静态导入库以使用 Windows API，一个 Windows 本地版本的 GNU 的调试器，以及其它多种工具。
>
> Mingw-w64 可运行于本地 Microsoft Windows 平台，"cross-native"在 MSYS2 或 Cygwin。Mingw-w64 能生成 32-或 64-位可执行程序，运行于 `i686-w64-mingw32` 或 `x86_64-w64-mingw32` 目标平台。

访问[SourceForge MinGW-w64](https://sourceforge.net/projects/mingw-w64/files/)，选择下方“MinGW-W64 GCC-8.1.0”中的“x86_64-posix-seh”，点击即可下载。

> 注：下载可能需要国际网络环境。

# 安装编译器 MinGW-w64

将下载好的 7z 压缩包解压，存放在 C 盘根目录下，记录 g++ 的绝对路径（如`C:\mingw64\bin`）。

右击“此电脑”，选择“属性”，选择“高级系统设置”，选择“环境变量”，在下方“系统变量”中找到“Path”，双击：

![环境变量1](https://i0.hdslb.com/bfs/album/ab86d6fe7d81d301576ae9b9ed1483d88a744572.png)

新建，填入记录的路径：

![环境变量2](https://i0.hdslb.com/bfs/album/84b31f5c6a71196376e86672392d6ce960be29f3.png)

逐级确定以保存。

验证。按下 `Win` + `R`，运行 `cmd`。

输入 `gcc` 回车，若提示 `gcc: fatal error: no input files`，则说明配置成功；若提示“‘gcc’不是内部或外部命令”，则说明环境变量添加失败；否则配置失败，应该按照上述操作重新配置。

输入 `gcc -v` 回车，可以查看 gcc 的版本。

# 配置 VS Code

## 安装扩展（Extensions）

推荐安装扩展：

- Chinese (Simplified) Language Pack for Visual Studio Code
- C/C++
- Code Runner

{% note warning %}

安装 Code Runner 后，如果需要在终端运行程序，需要在“设置”-“扩展”-“Run Code configuration”处打开“Run In Terminal”选项，并对应修改 `launch.json` 文件的相应位置。

{% endnote %}

其他推荐扩展：

- One Dark Pro
- Bracket Pair Colorizer
- Rainbow Brackets
- Code Spell Checker
- Git Graph
- Git History
- filesize
- Markdown All in One
- Markdown PDF
- markdownlint
- Tabnine Autocomplete AI: JavaScript, Python, TypeScript, PHP, Go, Java, Ruby, C/C++, HTML/CSS, C#, Rust, SQL, Bash, Kotlin, React
- x86 and x86_64 Assembly

## 配置 json 文件

创建工作区文件夹，用于存放代码，路径中不要出现中文、空格等符号，然后用 VS Code 打开此文件夹。

- 按下 `Ctrl`+`Shift`+`P` ，输入“C/C++”，选择“C/C++: Edit configurations (UI)”，可以发现文件夹下自动创建了子文件夹 `.vscode`，其中包含 `c_cpp_properties.json` 文件。

  参考代码：

  ```json
  {
    "configurations": [
      {
        "name": "Win32",
        "includePath": [
          "${workspaceFolder}/**",
          "C:\\mingw64\\lib\\gcc\\x86_64-w64-mingw32\\8.1.0\\include"
        ],
        "defines": ["_DEBUG", "UNICODE", "_UNICODE"],
        "compilerPath": "C:\\mingw64\\bin\\g++.exe",
        "cStandard": "c11",
        "cppStandard": "c++17",
        "intelliSenseMode": "${default}"
      }
    ],
    "version": 4
  }
  ```

  如图：

  ![c_cpp_propeerties_zyk](https://i0.hdslb.com/bfs/album/ebf9c88e7e20c020b2491b4817c45293aa48021b.png)

- 按下 `Ctrl`+`Shift`+`P` ，输入“tasks”，选择“Tasks: Configure Default Build Task”，选择创建 `tasks.json` 文件，选择“Others”，VS Code 会在`.vscode` 文件夹下创建 `tasks.json` 文件。

  参考代码：

  ```json
  // https://code.visualstudio.com/docs/editor/tasks
  {
      "version": "2.0.0",
      "tasks": [{
          "label": "Compile", // 任务名称，与launch.json的preLaunchTask相对应
          "command": "g++",   // 要使用的编译器，C用gcc
          "args": [
              "${file}",
              "-o",    // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
              "${fileDirname}/${fileBasenameNoExtension}.exe",
              "-g",    // 生成和调试有关的信息
              "-m64",  // 不知为何有时会生成16位程序而无法运行，此条可强制生成64位的
              "-Wall", // 开启额外警告
              "-static-libgcc",     // 静态链接libgcc，一般都会加上
              "-fexec-charset=GBK", // 生成的程序使用GBK编码，不加这条会导致Win下输出中文乱码；繁体系统改成BIG5
          ], // 编译的命令，其实相当于VSC帮你在终端中输了这些东西
          "type": "process", // process是把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
          "group": {
              "kind": "build",
              "isDefault": true // 不为true时ctrl shift B就要手动选择了
          },
          "presentation": {
              "echo": true,
              "reveal": "always", // 执行任务时是否跳转到终端面板，可以为always，silent，never。具体参见VSC的文档，即使设为never，手动点进去还是可以看到
              "focus": false,     // 设为true后可以使执行task时焦点聚集在终端，但对编译C/C++来说，设为true没有意义
              "panel": "shared"   // 不同的文件的编译信息共享一个终端面板
          },
          "problemMatcher":"$gcc" // 捕捉编译时终端里的报错信息到问题面板中，修改代码后需要重新编译才会再次触发
          // 本来有Lint，再开problemMatcher就有双重报错，但MinGW的Lint效果实在太差了；用Clangd可以注释掉
      }]
  }
  ```

  如图：

  ![tasks_zyk](https://i0.hdslb.com/bfs/album/cba32e9fa3ea13a9dee468ccd322926e11544aad.png)

- 按下 `Ctrl`+`Shift`+`P` ，输入“launch”，选择“Debug: Open launch.json”，VS Code 会在`.vscode` 文件夹下创建 `launch.json` 文件。

  参考代码：

  ```json
  // https://code.visualstudio.com/docs/cpp/launch-json-reference
  {
      "version": "0.2.0",
      "configurations": [{
          "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
          "type": "cppdbg", // 配置类型，对于C/C++可认为此处只能是cppdbg，由cpptools提供；不同编程语言不同
          "request": "launch", // 可以为launch（启动）或attach（附加）
          "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径，或"${workspaceFolder}\\${fileBasenameNoExtension}.exe"
          "args": [], // 程序调试时传递给程序的命令行参数，一般设为空
          "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点，或true
          "cwd": "${workspaceFolder}", // 调试程序时的工作目录，此为工作区文件夹；改成${fileDirname}可变为文件所在目录
          "environment": [], // 环境变量
          "externalConsole": true, // 使用单独的cmd窗口，与其它IDE一致；为false时使用内置终端，或false
          "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
          "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb。但我没试过lldb
          "miDebuggerPath": "C:\\mingw64\\bin\\gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要
          "setupCommands": [
              { // 模板自带，好像可以更好地显示STL容器的内容，具体作用自行Google
                  "description": "Enable pretty-printing for gdb",
                  "text": "-enable-pretty-printing",
                  "ignoreFailures": false // 或true
              }
          ],
          "preLaunchTask": "Compile" // 调试前执行的任务，一般为编译程序。与tasks.json的label相对应
      }]
  }
  ```

  如图：

  ![launch_zyk](https://i0.hdslb.com/bfs/album/4c3a63cb1bc0eb1a48e78733422430119e29baa8.png)

# 运行 C++ 程序

在工作区文件夹内，`.vscode` 文件夹外，新建一个 `.cpp` 文件，运行以检验 VS Code 配置成果。
