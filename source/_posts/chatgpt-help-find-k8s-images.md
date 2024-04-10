---
title: chatgpt-help-find-k8s-images
date: 2024-03-27 14:11:24
tags:
---

# Kubectl查询脚本

## 用户:
帮我写一个脚本使用 kubectl 查询集群上所有 pod image 以 docker.foobar.com 开头的镜像列表，去重排序

## ChatGPT:
要编写一个脚本来查询 Kubernetes 集群上所有以 `docker.foobar.com` 开头的 Pod 镜像，并进行去重排序，我们可以使用 `kubectl` 命令行工具和一些文本处理工具，如 `awk`, `sort`, `uniq` 等。

以下是一个简单的 Bash 脚本示例，它将执行这些操作：

```bash
#!/bin/bash

# 查询集群上所有 pod 的镜像信息
images=$(kubectl get pods --all-namespaces -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n')

# 过滤出以 docker.foobar.com 开头的镜像，并去重排序
echo "$images" | grep '^docker\.foobar\.com' | sort | uniq

```

这个脚本的工作流程如下：

1. 使用 `kubectl get pods --all-namespaces -o jsonpath="{..image}"` 获取集群中所有命名空间的所有 Pod 的镜像信息。
2. 使用 `tr -s '[[:space:]]' '\n'` 将输出中的空格转换为换行符，这样每个镜像都在单独的一行。
3. 使用 `grep '^docker\.foobar\.com'` 来过滤出以 `docker.foobar.com` 开头的镜像。
4. 使用 `sort` 对结果进行排序。
5. 使用 `uniq` 命令去除重复的镜像。

要运行这个脚本，你需要在一个有 `kubectl` 访问权限的环境中执行以下步骤：

1. 将上面的脚本保存为一个文件，例如 `get-foobar-images.sh`。
2. 通过运行 `chmod +x get-foobar-images.sh` 命令使脚本可执行。
3. 运行脚本 `./get-foobar-images.sh`。

请确保你的 `kubectl` 配置正确，并且你有足够的权限来查询所有命名空间中的 Pods。

## 用户:
一行实现

## ChatGPT:
可以使用一条命令行语句来完成这个任务。在 shell 中，您可以使用管道（`|`）来连接多个命令，形成一个单一的命令行。

```sh
kubectl get pods --all-namespaces -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | grep '^docker\.foobar\.com' | sort -u
```

这条命令解释如下：

- `kubectl get pods --all-namespaces -o jsonpath="{..image}"`：获取所有命名空间中的 Pods 的镜像信息。
- `tr -s '[[:space:]]' '\n'`：将输出中的空格和换行符压缩成单个换行符，以便每个镜像占一行。
- `grep '^docker\.foobar\.com'`：过滤出以 `docker.foobar.com` 开头的镜像。
- `sort -u`：对结果进行排序，并使用 `-u`（唯一）选项去除重复的行。

您可以直接在终端中运行这条命令，前提是您已经配置了对应的 `kubectl` 访问权限。