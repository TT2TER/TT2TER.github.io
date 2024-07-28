---
title: 最简单的内网穿透方法！Tailscale
tags:
  - 内网穿透
  - ipv6
  - Tailscale
  - network
categories:
  - 教程
  - share
excerpt: 解决的需求，在没有公网跳板机的情况下访问无法外网访问的服务器。
date: 2024-07-28 13:39:09
---


# 很久没有写blog了
很久没有写博客了，最近半年都比较忙，很多时候都没有时间写杂七杂八的东西，在这里交代一下近况，如果没有兴趣，请直接从侧边导航栏跳转到正文部分。

最近进组了，搞了一块很顶的4090显卡，在实验室工位上。最近刚刚把系统驱动环境搞定（真的挺不容易的，详情可以从主板上碰掉的电容说起……）。有了这么顶的显卡，当然要随时随地跑代码喽，那么外网访问就是必须要解决的事情。尤其是linux系统在校园网环境下拿不到ipv6老毛病一直没有得到解决，根本解决不了好吧。我还只有一个公网的仅ipv6服务器。想自己搭内网穿透都麻烦的很。找了很多服务，现成的内网穿透服务商，以及cloudflare，甚至是用warp给linux拿一个ipv6的方法都考虑过，都不是很好用。

在和神奇的前舍友讨论之后，找到了这个简单而且稳定的服务，[tailscale](https://tailscale.com/)

# 正文

用法很简单，访问官网，点击get started，登陆账号（我用的github），跟着教程一步一步的来，先添加两个设备。

如果不放心，可以看看这个[quick-start教程](https://tailscale.com/kb/1017/install)

为你所有想安装的服务安装上之后，就可以直接使用了

## 最佳实践

### 为设备改名
将设备名称改为更短更好记忆的名称

在控制面板的machines中找到你想改名的设备，在右侧点击三个点，并且编辑设备名称。

（用处，这样可以在ping/ssh/访问设备中的服务的时候直接用这个短的名称作为设备的名字）

注意，如果名称是纯数字，会在ping等命令中出现问题。

### 使用magic dns
更改tailent name

可以在dns选项中将域名改为foo-ber.ts.net的好记忆的域名

foo bar是两个随机分配的英文单词，有的组合还是很有意思的

在命名完之后，就可以用<machines-name>.<tailent name>来唯一确定你的任意组网内的机器了。

参考[magic-dns](https://tailscale.com/kb/1081/magicdns)

当然，如果你自己有短域名或者更顺手的域名，可以用自己的的域名解析dns，但是就得在你的域名管理平台自己添加了

### 打开tailscale自己的ssh

可以通过tailscale连接ssh,免去输入密码/配置密钥的困扰（当然正常的ssh连接的方案不受影响）

我还没有阅读他的详细的原理，但是这个方法看起来（尝试起来）都很好用

参考[ssh](https://tailscale.com/kb/1193/tailscale-ssh)

ssh连接命令示例
```bash
ssh ubuntu@device
```

### 剩下的功能没有尝试，也暂时没有需求

[尝试了一下出口节点（感觉就像代理一样，组网内的设备可以选择从出口节点出站)](https://tailscale.com/kb/1103/exit-nodes)

我用宿舍的工控机和手机尝试了一下，不是很好用，流量没有转发。因为没有需求，因此没有进一步尝试。

### 建议配合tmux等工具一起使用，断网不心疼

tmux 中zsh没有颜色解决方案

非官方方法
```shell
echo "export TERM=xterm-256color" >> ~/.zshrc
```

[参考博客](https://www.mojidong.com/post/2017-05-14-zsh-autosuggestions/)

[相关讨论](https://unix.stackexchange.com/questions/139082/zsh-set-term-screen-256color-in-tmux-but-xterm-256color-without-tmux)