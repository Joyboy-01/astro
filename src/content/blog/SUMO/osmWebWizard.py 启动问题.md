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