---
title: 关闭 Linux 图形化界面以节省内存
tags:
  - linux
  - gui
categories:
  - 教程
excerpt: 宿舍的工控机只用来跑一些服务，不需要图形化界面，关闭图形化界面可以节省内存，提高性能
date: 2025-01-11 16:53:34
---

![关闭后内存消耗急剧下降](https://pic.1314171.xyz/i/2025/01/09/20250109154847804.png)
关闭后内存消耗急剧下降
# 临时关闭：
运行以下命令停止显示管理器：
```bash
sudo systemctl stop display-manager
```

系统会切换到命令行模式，此时图形界面相关进程被停止，节省内存。

# 永久关闭：
修改系统默认启动目标为命令行模式：
```bash
sudo systemctl set-default multi-user.target
sudo reboot

#如果需要重新启用图形化界面：

sudo systemctl set-default graphical.target
sudo reboot
```

关闭图形界面后会影响哪些功能？
	1.	无法使用桌面应用：
	•	图形化应用（如浏览器、文本编辑器等）无法运行，但命令行工具仍可正常使用。
	2.	远程桌面功能中断：
	•	如果依赖 VNC、NoMachine 等远程图形工具，它们将无法工作。不过，SSH 和命令行工具不受影响。
	3.	HDMI 显示屏输出：
	•	HDMI 可能只显示命令行，而非图形界面。