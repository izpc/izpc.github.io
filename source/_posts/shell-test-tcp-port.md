---
title: shell-test-tcp-port
date: 2024-09-20 17:59:14
tags:
---

# 测试 TCP 端口 3306 是否开放

要测试 3306 端口的连通性，可以使用多种 Linux 命令工具，如 `telnet`、`nc`（netcat）或 `nmap`。以下是几种常用的方法：

### 使用 `telnet`
`telnet` 是一个简单的命令行工具，可以用于测试指定主机和端口的连通性。
```sh
telnet <目标IP> 3306
```
如果连接成功，会显示类似于 `Connected to <目标IP>` 的消息。如果连接失败，会显示 `Unable to connect to remote host`。

### 使用 `nc`（netcat）
`nc` 是一个更强大的网络工具，可以用于测试端口连通性。
```sh
nc -zv <目标IP> 3306
```
- `-z` 选项表示扫描端口而不发送数据。
- `-v` 选项表示显示详细信息。

如果连接成功，会显示 `succeeded`，否则会显示 `failed`。

### 使用 `nmap`
`nmap` 是一个网络扫描工具，可以用于检查端口的开放状态。
```sh
nmap -p 3306 <目标IP>
```
这将扫描目标主机的 3306 端口，并显示端口的状态（open/closed/filtered）。

### 使用 `curl`
如果你想测试 MySQL 服务是否在运行，可以使用 `curl` 发送一个简单的 HTTP 请求（假设 MySQL 服务配置了 HTTP 接口）。
```sh
curl http://<目标IP>:3306
```
这通常不会成功，因为 MySQL 默认不提供 HTTP 接口，但它可以用于测试是否有任何响应。

### 示例
假设目标 IP 是 `192.168.1.100`，可以使用以下命令：

#### 使用 `telnet`
```sh
telnet 192.168.1.100 3306
```

#### 使用 `nc`
```sh
nc -zv 192.168.1.100 3306
```

#### 使用 `nmap`
```sh
nmap -p 3306 192.168.1.100
```

选择适合你的工具和方法来测试 3306 端口的连通性。如果你没有安装这些工具，可以使用包管理器进行安装，例如 `apt-get`、`yum` 或 `brew`。