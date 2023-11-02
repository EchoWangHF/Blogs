vscode 舒适的配置
```
{
    "editor.minimap.enabled": false,
    "editor.parameterHints.enabled": false,
    "editor.quickSuggestions": {
      "comments": "on",
      "strings": "on",
      "other": "on"
    },
    "editor.fontSize": 15,
    "editor.wordWrap": "on",
    "[python]": {
      "editor.formatOnType": true
    },
    "editor.inlineCompletionsAccessibilityVerbose": true,
}
```



作为在Windows环境下习惯使用Visual Studio IDE的人，对于Linux环境下的Vim编辑使用十分难受，虽然网上很多人说vim非常牛逼和强大，但是我更加习惯于使用VS code的界面，所以我选择VS code作为编辑器使用。

VS code本身是一个编辑器，所以如果需要调试等功能需要自己安装一些插件，并且配置相关的json文件。

linux 环境下，g++和clang都可以作为C++的编译器，我这里选择使用的是clang。

首先是插件选择：

(1) C/C++ 微软自带的C/C++插件。

(2) C/C++ Clang Command Adapter：提供静态检测（Lint）

(3) Code Runner：右键即可编译运行单文件

(4) Bracket Pair Colorizer：彩虹花括号

(5) Include Autocomplete：提供头文件名字的补全

 

以上插件下载完之后，在文件工作区(workspace) 新建一个文件夹作为你项目的根目录文件，然后新建一个".vscode" 文件，该文件夹存放相关json的配置文件，其中launch.json 和tasks.json 两个配置文件是必须的，除了这两个之外，还可以加上setting.json。如果setting.json不加设置，就会使用设置，基本上满足使用了。

关于，launch.json的配置为：

```c
// https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，这里只能为cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${fileDirname}/${fileBasenameNoExtension}.out", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，我一般设置为true
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录
            "environment": [], // （环境变量？）
            "externalConsole": true, // 调试时是否显示控制台窗口，一般设置为true显示控制台
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb。但目前lldb在windows下没有预编译好的版本。
            // "miDebuggerPath": "gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则去掉
            "setupCommands": [ // 用处未知，模板如此
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ],
            "preLaunchTask": "Compile" // 调试会话开始前执行的任务，一般为编译程序。与tasks.json的label相对应
        }
    ]
}
```
　tasks.json的配置为：
 ```c
 // https://code.visualstudio.com/docs/editor/tasks
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile", // 任务名称，与launch.json的preLaunchTask相对应
            "command": "clang++", // 要使用的编译器 clang++  或者g++
            "args": [
                "${file}",
                "-o", // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
                "${fileDirname}/${fileBasenameNoExtension}.out",
                "-g", // 生成和调试有关的信息
                "-Wall", // 开启额外警告
                "-static-libgcc", // 静态链接
                // "-fcolor-diagnostics", // 彩色的错误信息？但貌似clang默认开启而gcc不接受此参数
                // "--target=x86_64-w64-mingw", // clang的默认target为msvc，不加这一条就会找不到头文件；Linux下去掉这一条
                "-std=c++11" // C语言最新标准为c11，或根据自己的需要进行修改
            ], // 编译命令参数
            "type": "shell", // 可以为shell或process，前者相当于先打开shell再输入命令，后者是直接运行命令
            "group": {
                "kind": "build",
                "isDefault": true // 设为false可做到一个tasks.json配置多个编译指令，需要自己修改本文件，我这里不多提
            },
            "presentation": {
                "echo": true,
                "reveal": "always", // 在“终端”中显示编译信息的策略，可以为always，silent，never。具体参见VSC的文档
                "focus": false, // 设为true后可以使执行task时焦点聚集在终端，但对编译c和c++来说，设为true没有意义
                "panel": "shared" // 不同的文件的编译信息共享一个终端面板
            }
            // "problemMatcher":"$gcc" // 如果你不使用clang，去掉前面的注释符，并在上一条之后加个逗号。照着我的教程做的不需要改（也可以把这行删去)
        }
    ]
}
 ```
 
 然后ctrl+shift+B是编译，按F5是编译+运行。

以上是VS Code在Linux环境下使用的基本操作，日后有什么新的技能和发现我会更新或者修正。
