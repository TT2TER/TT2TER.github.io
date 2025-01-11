---
title: 针对 SMB 的速度测试，并优化传输速度
tags:
  - smb
  - iptables
categories:
  - 教程
excerpt: 工位的电脑在校园网里又加了层nat，导致smb传输速度很慢，这里记录一下解决方法
date: 2025-01-11 16:50:08
---


# 问题描述和分析
工位上的电脑和家里的nas使用之前提到的tailscale部署在了同一个局域网内，但是smb传输速度很慢，大概只有几十kb/s，而且不稳定，有时候会掉到几kb/s，甚至断联。同时，连接时延也很高，ping延迟在500ms以上。

而宿舍的工控机，或者别的校园网环境内的电脑，使用smb传输速度都很快，可以达到10mb/s以上的下载速度。ping延迟也只有几十ms。甚至几ms。

## smb传输速度检测

创建测试文件
```bash
dd if=/dev/zero of=testfile bs=1m count=1024
```

挂载nas
```bash
# 在Ubuntu上挂载nas
sudo apt update
sudo apt install cifs-utils

sudo mkdir -p /mnt/smb
sudo mount -t cifs //<server>/<share> /mnt/smb -o username=<your-username>,password=<your-password>,vers=3.0
```

使用dd测试传输速度
```bash
# 上传到nas
dd if=testfile of=/mnt/smb/testfile bs=1M #可能会有权限问题，可以在nas上先创建这个文件
# 从nas下载
dd if=/mnt/smb/testfile of=/dev/null bs=1M

❯ 1024+0 records in
❯ 1024+0 records out
❯ 1073741824 bytes (1.1 GB, 1.0 GiB) copied, 90.8313 s, 11.8 MB/s
```
> 如果在mac上测试，使用dd命令时，需要使用`bs=1m`，而不是`bs=1M`，否则会报错`dd: bs: illegal numeric value`

比较不同网络环境的速度

取消挂载，删除测试文件
```bash
sudo umount /mnt/smb
sudo rmdir /mnt/smb
rm testfile
```

# 解决方法：使用iptables优化smb传输路径

网络拓扑假设

假设：
	•	A 的 IP 地址：192.168.1.10
	•	B 的 IP 地址：192.168.1.20
	•	C 的 IP 地址：192.168.1.30
	•	SMB 服务运行在 C 的 445 端口。

目标：
	•	A 通过访问 B 的 445 端口间接访问 C 的 SMB 服务。
	•	配置完成后，A 访问 192.168.1.20:445 相当于访问 192.168.1.30:445。

## 1. 在 B 上启用ip转发

```bash
# 临时开启
sudo sysctl -w net.ipv4.ip_forward=1
# 永久开启
sudo vim /etc/sysctl.conf
将其中的net.ipv4.ip_forward=1取消注释

# 重新加载配置
sudo sysctl -p
```

验证是否开启了ip转发
```bash
cat /proc/sys/net/ipv4/ip_forward
# 如果返回1，则表示开启了ip转发
```

这一步是确保B能够转发流量

## 2. 在 B 上配置iptables规则

### 添加PREROUTING规则
在B上添加PREROUTING规则，将所有访问B的445端口的流量转发到C的445端口
```bash
sudo iptables -t nat -A PREROUTING -p tcp -d 192.168.1.20 --dport 445 -j DNAT --to-destination 192.168.1.30:445
```
### 配置FORWARD规则
```bash
sudo iptables -A FORWARD -p tcp -s 192.168.1.10 -d 192.168.1.30 --dport 445 -j ACCEPT
sudo iptables -A FORWARD -p tcp -s 192.168.1.30 -d 192.168.1.10 --sport 445 -j ACCEPT
```
允许A和C之间的445端口的通信

### 配置POSTROUTING规则
```bash
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.30 --dport 445 -j MASQUERADE
```
这一步是确保C能够回复流量, 如果不配置这一步，C会自己直接回复给A，而不经过B，导致连接失败（当然也可以配置C的路由表，让C知道A的存在）

但做源地址伪装就不会有这个问题，C回复的数据包会被B伪装成B的数据包，A也会认为自己是在和B通信

## 3. 验证配置是否生效

可以使用之前的dd命令测试一下速度，如果速度有明显提升，说明配置生效了

经过我的测试，速度从0kb，没错，9kb/s，提升到了3mb/s，速度提升了300多倍！可以正常打开和保存文件，甚至可以正常播放视频。
![可以看到效果很好](https://pic.1314171.xyz/i/2025/01/09/20250109235227637.png)

虽然传输极大文件的速度还是不够理想，但是对于小文件的传输已经足够了。说明中转这一下速度影响还是很大的。（在宿舍时到nas的速度不中转也能达到10mb，而中转8mb）

如果有更大文件传输需求，可能需要考虑其他方法了……

# 持久化iptables规则

因为规则是临时的，重启后会失效，同时其他服务如tailscale，docker好像也会修改iptables规则，所以需要持久化规则，但又不能直接粗暴的`iptables-save > /etc/iptables/rules.v4`，因为这样会把其他规则也保存下来，可能会导致其他服务出问题。

所以需要将规则命令保存到一个脚本中，然后在开机时执行这个脚本。

```bash
# 创建启动脚本
sudo vim /usr/local/bin/iptables-setup.sh
```

将需要执行的命令写入脚本中
```bash
#!/bin/bash

# 添加 iptables 规则
iptables -t nat -A PREROUTING -p tcp -d 192.168.1.20 --dport 445 -j DNAT --to-destination 192.168.1.30:445

# 配置FORWARD规则
iptables -A FORWARD -p tcp -s 192.168.1.10 -d 192.168.1.30 --dport 445 -j ACCEPT
iptables -A FORWARD -p tcp -s 192.168.1.30 -d 192.168.1.10 --sport 445 -j ACCEPT
# 配置POSTROUTING规则
iptables -t nat -A POSTROUTING -p tcp -d 192.168.1.30 --dport 445 -j MASQUERADE
```

给脚本添加执行权限
```bash
sudo chmod +x /usr/local/bin/iptables-setup.sh
```

创建systemd服务
```bash
sudo vim /etc/systemd/system/iptables-setup.service
```

写入服务配置
```bash
[Unit]
Description=Setup iptables rules
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/iptables-setup.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

重新加载配置
```bash
sudo systemctl daemon-reload
```

启用服务
```bash
sudo systemctl enable iptables-setup.service
sudo systemctl start iptables-setup.service
sudo systemctl status iptables-setup.service
```