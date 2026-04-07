---
title: 断电后的 Ubuntu 急救全纪录
publishDate: 2026-4-08 01:18:00
description: 'Ubuntu 24.04 LTS'
tags:
  - linux
language: '中文'
---
一台 Windows + Ubuntu 双系统笔记本，电池耗尽强制关机后，重新开机发现：

- **Ubuntu**：内置屏幕黑屏，左上角只有一个光标在闪烁，系统无响应
- **Windows**：完全正常，没有任何问题
- 尝试连接外接显示器后，Ubuntu 的桌面可以在外接屏上正常显示，但没有 Wi-Fi，设置里连"网络"选项都不存在

---

## 观察错误现象

用外接屏进入系统后，打开终端，发现大量报错：

```
E: 无法下载 http://mirrors.aliyun.com/ubuntu/pool/restricted/n/
   nvidia-graphics-drivers-570/...  暂时不能解析域名 "mirrors.tencent.com"

E: 有几个软件包无法下载，要不运行 apt-get update 或者加上 --fix-missing 再试？

Failed to allocate directory watch: Too many open files
```

同时打开"软件和更新 → 附加驱动"，看到显卡是 **NVIDIA RTX 3060 Mobile**，当前使用的是 `nvidia-driver-580-open`（推荐驱动），驱动选择本身没有问题。

运行 `sudo apt-get update`，所有镜像源全部报**域名解析失败**，说明此时完全没有网络。

---

## 为什么 Windows 好好的，Ubuntu 却出问题？

第一反应是"硬件坏了"或者"系统文件损坏"，但仔细想想：

**Windows 和 Ubuntu 是完全独立的两套系统，住在各自独立的分区里。**

| | Windows | Ubuntu |
|---|---|---|
| 分区 | C盘（NTFS） | 独立分区（ext4） |
| 内核 | Windows NT 内核 | Linux 内核 |
| 驱动 | 独立的一套 | 独立的一套 |
| 更新 | 独立管理 | 独立管理 |

断电对 Windows 没影响，说明**磁盘没有损坏、硬件没有问题**。Ubuntu 出问题，只能是 Ubuntu 自己的某个文件或配置出了问题。

而且两个故障（内屏黑屏 + Wi-Fi 消失）**同时发生**，不像是随机损坏，更像是某一次操作被中断导致的连锁反应。

---

## 排查是驱动问题还是硬件问题

**关键线索：外接屏幕可以正常显示桌面。**

如果是显卡硬件坏了，外接屏也不会有画面。既然外接屏正常，说明系统已经成功启动，NVIDIA 显卡也在工作，只是内屏没有输出信号。

这台笔记本是**双显卡**结构：
- Intel 核显 → 负责内置屏幕输出
- NVIDIA 独显 → 负责外接屏幕和高性能计算

内屏黑屏，最可能的原因是 Intel 核显的驱动或配置出了问题，或者显卡切换模式异常。

先检查显卡切换模式：

```bash
sudo prime-select query
# 输出：on-demand
```

`on-demand` 是正确的模式，不是切换配置的问题。

再检查 Wi-Fi 网卡是否被系统识别：

```bash
lspci | grep -i network
# 输出：03:00.0 Network controller: MEDIATEK Corp. MT7922 802.11ax PCI Express Wireless Network Adapter
```

**网卡硬件是存在的**，系统能识别到它，只是没有驱动在跑。这排除了网卡硬件损坏的可能。

尝试手动加载 MT7922 对应的驱动模块：

```bash
lsmod | grep mt79       # 无任何输出，模块未加载

sudo modprobe mt7921e
# FATAL: Module mt7921e not found in directory /lib/modules/6.17.0-20-generic
```

报错很直白：**在当前内核（6.17.0-20）的模块目录里，根本找不到这个驱动文件**。不是驱动没有加载，是驱动文件压根不存在。

继续查这个内核目录里有没有任何 MT79 系列的驱动：

```bash
find /lib/modules/$(uname -r) -name "mt79*"
# 无任何输出
```

空的。整个 6.17.0-20 内核的无线驱动文件全部缺失。

---

## 锁定根本原因 —— 内核更新被断电打断

到这里，所有线索指向同一个方向：**当前运行的内核（6.17.0-20）安装不完整。**

在 Ubuntu 里，一次完整的内核更新需要安装多个包：

```
linux-image-6.17.0-20-generic        # 内核主体
linux-modules-6.17.0-20-generic      # 基础模块
linux-modules-extra-6.17.0-20-generic  # 扩展模块（含Wi-Fi驱动）
linux-headers-6.17.0-20-generic      # 内核头文件（NVIDIA驱动编译需要）
```

断电发生时，内核主体装完了（所以系统能启动进 6.17.0-20），但后续的 `modules-extra` 和 `headers` 没有装完。

- **`modules-extra` 缺失** → MT7922 的驱动文件不存在 → Wi-Fi 消失
- **`headers` 缺失** → NVIDIA 驱动的 `.ko` 文件无法编译 → 内屏输出异常

验证这个判断，查看已安装的 modules-extra：

```bash
dpkg -l | grep linux-modules | grep extra
```

6.17.0-20 对应的 extra 包显示为**未完整配置状态**，完全印证了推断。

---

## 找到救援工具 —— 旧内核

既然 6.17.0-20 是新安装的，那么旧内核 **6.17.0-19** 应该还在，而且是完整的。重启进入 GRUB 菜单验证：

```
GRUB 高级选项：
  Ubuntu, Linux 6.17.0-20-generic   ← 有问题的新内核
  Ubuntu, Linux 6.17.0-19-generic   ← 完整的旧内核
```

选择 **6.17.0-19** 启动，进入系统后 Wi-Fi 立刻出现，内屏也正常了。**硬件完好无损，问题 100% 是新内核的驱动文件缺失。**

---

## 在旧内核下修复新内核

有了网络，修复就很简单。在 6.17.0-19 内核下执行：

```bash
# 1. 补装 6.17.0-20 缺失的内核头文件
sudo apt install linux-headers-6.17.0-20-generic

# 2. 触发所有未完成包的后期配置（包括 NVIDIA 驱动的 .ko 文件编译）
sudo dpkg --configure -a

# 3. 修复依赖链上的残缺包
sudo apt --fix-broken install

# 4. 更新引导配置
sudo update-grub

# 5. 重启，选新内核验证
sudo reboot
```

安装 headers 之后，`dpkg --configure -a` 自动触发了 NVIDIA 驱动模块的编译，终端输出了一系列 `.ko` 文件的链接过程，之前报错的 `nvidia-drm.ko`、`nvidia.ko` 等文件全部生成成功。

重启后进入 6.17.0-20，内屏正常，Wi-Fi 正常，故障完全解除。

---

## 整体故障链梳理

```
电池耗尽 → 强制断电
    ↓
Ubuntu 内核更新被打断（6.17.0-19 → 6.17.0-20）
    ↓
新内核主体装完，但 modules-extra 和 headers 未装完
    ↓
系统默认进入新内核 6.17.0-20
    ├── modules-extra 缺失 → mt7922 驱动不存在 → Wi-Fi 消失
    └── headers 缺失 → NVIDIA .ko 文件未编译 → 内屏无输出
    ↓
Windows 独立分区，完全不受影响
```

---

## 经验总结

- **外接屏能用就不是硬件问题**，内屏黑屏优先怀疑驱动，不要慌
- **Wi-Fi 消失先跑 `lspci`**，硬件能识别说明是驱动问题，而不是网卡坏了
- **双故障同时出现**，大概率是同一个根因，不要分开处理，找共同点
- **Windows 正常不代表硬件没问题**，但能证明磁盘和基本硬件是好的，大大缩小排查范围
- **GRUB 里的旧内核是最强救援工具**，不需要 U盘、Live CD 或任何外部资源，优先考虑
- Ubuntu 后台更新期间尽量不要强制断电，如果必须断电，下次开机后第一件事是运行 `sudo dpkg --configure -a && sudo apt --fix-broken install`