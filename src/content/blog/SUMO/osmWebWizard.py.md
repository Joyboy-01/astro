---
title: osmWebwizard.py 
publishDate: 2025-11-05 16:18:00
description: 'OMNET++ debug问题'
tags:
  - SUMO
  - linux
language: '中文'
---

模拟交通环境要在线生成路网，之前sumo1.24.0没什么问题，但是由于VEINS对SUMO的版本有需求，所以在`apt-cache madison sumo`找到了sumo1.18
然而在使用osmWebWizard.py文件的时候无法找到Wizard包，可能是文件缺失的问题。
为了方便就直接把源码的文件夹放过来了

```shell
# 1. 进入您的“下载”目录
cd ~/下载

# 2. 下载 SUMO 1.18.0 版本的源代码压缩包
wget https://github.com/eclipse/sumo/archive/refs/tags/v1_18_0.zip
# 1. 解压刚刚下载的文件
unzip v1_18_0.zip

# 2. 此时，您会得到一个名为 sumo-1.18.0 的文件夹
# 我们可以确认一下缺失的 webwizard 模块确实在里面：
ls ~/下载/sumo-1.18.0/tools/webWizard
# 使用 sudo (管理员权限) 来复制文件夹
sudo cp -r ~/下载/sumo-1.18.0/tools/webWizard /usr/share/sumo/tools/
```
如果之后还有问题，就把这个包编译安装一下当库好了

---
第二个问题，Ubuntu24.04在sumo1.8.0版本的下载时总会编译不通过，好像是因为系统库（如 libgdal 和 libhdf5）要求链接一个特定版本的 libcurl（OpenSSL 版本）。然而，在编译 SUMO 时，链接器 (ld) 没有被明确告知要去搜索 libcurl 这个库，导致它找不到这些函数。即使我们已经确保了 libcurl4-openssl-dev 已安装，CMake 也可能因为“传递依赖”链条断裂而忽略它。

在重新运行 CMake 时，强制将 libcurl 添加到最终可执行文件的链接指令中。

```
cmake .. -DCMAKE_EXE_LINKER_FLAGS="-lcurl"
```