---
title: Ubuntu 设置系统时钟同步
date: 2024-05-16 16:12:29
tags: [ubuntu, linux, ntp, systemd]
---

在Ubuntu系统中，您可以通过`timedatectl`命令或者安装和配置`ntp`服务来设置时钟同步。

### 使用`timedatectl`命令

1. 首先，您可以使用`timedatectl`命令查看当前的时间同步状态：

```bash
timedatectl status
```

2. 如果`NTP enabled`没有被激活，您可以使用以下命令来激活NTP时间同步：

```bash
sudo timedatectl set-ntp true
```

这将会启用`systemd-timesyncd`服务，该服务会与外部NTP服务器同步时间。

### 安装和配置NTP服务

如果您需要更精细的时间同步控制，或者`systemd-timesyncd`不满足您的需求，您可以安装完整的NTP服务。

1. 安装NTP服务：

```bash
sudo apt-get update
sudo apt-get install ntp
```

2. 安装完成后，编辑NTP配置文件`/etc/ntp.conf`，您可以使用`nano`或者`vi`编辑器：

```bash
sudo nano /etc/ntp.conf
```

或者

```bash
sudo vi /etc/ntp.conf
```

3. 在配置文件中，您可以添加或修改NTP服务器。例如，您可以使用公共NTP服务器或您组织内部的NTP服务器。

```conf
server 0.ubuntu.pool.ntp.org
server 1.ubuntu.pool.ntp.org
server 2.ubuntu.pool.ntp.org
server 3.ubuntu.pool.ntp.org
```

4. 保存并关闭文件。然后重启NTP服务以应用更改：

```bash
sudo systemctl restart ntp
```

5. 最后，您可以使用以下命令检查NTP服务的状态：

```bash
ntpq -p
```

这将显示一个列表，其中包含NTP服务器和它们的状态。

确保您的防火墙或网络安全设置允许NTP服务的流量（通常是UDP 123端口）。如果您在一个受限制的网络环境中，您可能需要配置特定的NTP服务器或调整防火墙设置。