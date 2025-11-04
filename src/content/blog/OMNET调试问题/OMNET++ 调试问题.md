---
title: OMNET++ debug问题
publishDate: 2025-07-08 15:29:00
description: 'OMNET++ debug问题'
tags:
  - OMNET
  - linux
language: '中文'
---

环境 Ubuntu 24.04 ，在 omnetpp 目录文件夹下进入 OMNET。

![进入 OMNeT++ 目录](./pic/Pasted%20image%2020251104134559.png)

---

在学 OMNeT++ 的 tictoc part2 就遇到了困难，在 debug 模式下运行就会卡住不动，GUI 界面没有反应，需要强制退出。

![运行卡住界面](./pic/%E6%88%AA%E5%9B%BE%202025-11-04%2001-57-23.png)

---

据说是 Ubuntu 的问题，图形化界面总是兼容性不好，lldb 和 gdb 都会卡住。  
现在的解决办法是在 IDE 的 Terminal 里面使用终端版 gdb/lldb 调试。  
在卡住的时候用 `c` 步进，神奇的是 GUI 也会跟着动，就这样一卡一卡地凑合用。

![终端调试](./pic/Pasted%20image%2020251104135920.png)

---

gdb 则是要在这里配置一个新的：

![配置 gdb](./pic/Pasted%20image%2020251104150114.png)

之后运行后，在 **Window → Show View → Debugger Console** 启动调试器：

![启动调试器](./pic/Pasted%20image%2020251104150412.png)

---

或者也可以直接在命令行启动：

```bash
omnetpp-6.2.0:~/omnetpp_ws/tictoc$ gdb --args ./tictoc_dbg -u Qtenv -n . omnetpp.ini
```

![命令行运行](./pic/Pasted%20image%2020251104143347.png)
