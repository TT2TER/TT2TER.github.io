---
title: 在Linux上使用v2raya代理
tags:
  - v2raya
  - proxy
categories:
  - 教程
excerpt: 鉴于clash的作者已经不再维护clash，而且最近在用很多国内的linux服务器跑代码，所以就想着在linux上使用v2raya代理
date: 2024-01-31 22:01:59
---
# 2024-07-27更新
最近因为各种服务器都需要连网，因此自己写了一个可以在短期内很好的遵循的指导（tutorial）来配置v2raya代理，欢迎大家试用。debian系如Ubuntu，raspbian等都可以使用。

项目链接:[一个简易的不自动化的autodl部署自己的代理的指南，帮助下载huggingface的模型](https://github.com/TT2TER/autodl_proxy)

# 在windows上使用v2rayn代理即可
# 在Linux上使用v2raya代理
> 以Ubuntu为例（大家都喜欢！

## 安装
[安装v2raya](https://v2raya.org/docs/prologue/installation/debian/)

两种办法，一种是deb包，一种是直接apt下载，这两种方法都亲测可用

dab包可以留下来以供未来需要

官方说的很清楚

之前安装时尝试未成功应该是因为clash在运行而且我的clash被我调的占用了10809端口

以下是官方文档的备份，用不用安装xray根据自己机场的协议情况来

### 方法一：通过软件源安装

#### 添加公钥

```bash
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc
```

#### 添加 V2RayA 软件源

```bash
echo "deb [signed-by=/etc/apt/keyrings/v2raya.asc] https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
```

#### 安装 V2RayA

```bash
sudo apt install v2raya v2ray ## 也可以使用 xray 包
```

### 方法二：手动安装 deb 包

[从 Release 下载 v2rayA 的 deb 包](https://github.com/v2rayA/v2rayA/releases)后可以使用 Gdebi、QApt 等图形化工具来安装，也可以使用命令行：

```bash
sudo apt install /path/download/installer_debian_xxx_vxxx.deb ### 自行替换 deb 包所在的实际路径
```

V2Ray / Xray 的 deb 包可以在 [APT 软件源中](https://github.com/v2rayA/v2raya-apt/tree/master/pool/main/)找到。

## 启动 v2rayA / 设置 v2rayA 自动启动

> 从 1.5 版开始将不再默认为用户启动 v2rayA 及设置开机自动。

- 启动 v2rayA

  ```bash
  sudo systemctl start v2raya.service
  ```

- 设置开机自动启动

  ```bash
  sudo systemctl enable v2raya.service
  ```

- 好像不用systemctl启动也行，直接运行v2raya就行
    ```bash
    v2raya
    ```

## 使用方法
没啥大问题，摸索以下就行了