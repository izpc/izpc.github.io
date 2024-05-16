---
title: Shell 小技巧 —— Systemctl
date: 2024-05-16 16:05:52
tags: [shell, systemctl, linux, postgres, ChatGPT]
---


# 查看 systemctl 启动的服务列表

在 Linux 系统中，`systemctl` 是一个用于控制 `systemd` 系统和服务管理器的命令行工具。要查看由 `systemctl` 启动的服务列表，你可以使用以下命令：

1. 查看所有已启动的服务：

   ```sh
   systemctl list-units --type=service --state=running
   ```

   这个命令会列出所有当前处于运行状态的服务。

2. 查看所有已安装的服务，不论其状态：

   ```sh
   systemctl list-units --type=service --all
   ```

   使用 `--all` 选项可以显示所有服务，包括未启动或者处于不活跃状态的服务。

3. 查看特定服务的状态：

   ```sh
   systemctl status <service_name>.service
   ```

   将 `<service_name>` 替换为你想要查看状态的服务名。这会给出该服务的详细状态信息，包括是否正在运行、启动失败的原因等。

4. 查看所有服务的状态摘要：

   ```sh
   systemctl list-unit-files --type=service
   ```

   这个命令会列出所有服务的状态摘要，显示服务是否被设置为在启动时自动启动等信息。

5. 查看特定状态的服务：

   如果你只对特定状态的服务感兴趣，比如想要查看所有失败的服务，可以使用：

   ```sh
   systemctl --failed
   ```

   这将列出所有处于失败状态的服务。

6. 启用和禁用服务：

   如果你想要设置服务在启动时自动运行或者禁止自动运行，可以使用以下命令：

   - 启用服务：

     ```sh
     systemctl enable <service_name>.service
     ```

   - 禁用服务：

     ```sh
     systemctl disable <service_name>.service
     ```

   替换 `<service_name>` 为你想要启用或禁用的服务名。

记得在使用 `systemctl` 命令时，根据你的系统设置，你可能需要使用 `sudo` 来获取必要的权限。


# 如何关闭 postgres

如果你想要停止、禁用或者重启PostgreSQL服务，你可以使用`systemctl`命令。以下是一些常用的命令：

1. 停止PostgreSQL服务：
   ```bash
   sudo systemctl stop postgresql
   ```

2. 禁用PostgreSQL服务，使其不在启动时自动运行：
   ```bash
   sudo systemctl disable postgresql
   ```

3. 重启PostgreSQL服务：
   ```bash
   sudo systemctl restart postgresql
   ```

4. 查看PostgreSQL服务的状态：
   ```bash
   sudo systemctl status postgresql
   ```

如果你有多个PostgreSQL版本安装在你的系统上，服务名称可能会包含版本号，例如`postgresql-10`。你可以通过运行以下命令来查看所有的PostgreSQL服务：

```bash
sudo systemctl list-units --type=service | grep postgres
```

根据输出，你可能需要指定具体的服务名来停止或者禁用特定版本的PostgreSQL服务，例如：

```bash
sudo systemctl stop postgresql@10-main
sudo systemctl disable postgresql@10-main
```

替换`10`为你的PostgreSQL版本号。如果你不确定版本号，上面的命令将帮助你找到正确的服务名。