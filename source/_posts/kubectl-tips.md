---
title: kubectl 快捷命令
date: 2024-05-11 18:27:04
tags: [kubectl, k8s, kubernetes]
---

# 如何使用 kubectl 删除以 foo-bar 开头的 job


在 Kubernetes 中，如果你想要删除所有以特定前缀开头的 Job，你可以使用 `kubectl` 命令与 shell 的管道和过滤命令组合来完成这个操作。

下面是一个使用 `kubectl` 删除所有以 `foo-bar` 开头的 Job 的命令示例：

```sh
kubectl get jobs -o name | grep 'foo-bar' | xargs kubectl delete
```

这个命令的工作原理如下：

1. `kubectl get jobs -o name`：列出所有 Job 的名称。
2. `grep 'foo-bar'`：使用 `grep` 命令过滤出以 `foo-bar` 开头的 Job 名称。
3. `xargs kubectl delete`：`xargs` 命令会接收来自 `grep` 的输出，并将每一行输出作为参数传递给 `kubectl delete` 命令，从而删除这些 Job。

在执行这个命令之前，请确保你有足够的权限来删除 Job，并且你已经确认过这些 Job 是可以被删除的，因为该操作是不可逆的。如果你想要先看看具体会删除哪些 Job，你可以先运行不带 `xargs kubectl delete` 的命令：

```sh
kubectl get jobs -o name | grep 'foo-bar'
```

这将只列出所有将被删除的 Job 的名称，不会执行任何删除操作。如果你确认这些是你想要删除的 Job，再运行完整的删除命令。

如果你的 Kubernetes 集群中 Job 资源不是在默认命名空间中，你需要指定正确的命名空间，使用 `-n` 或 `--namespace` 参数：

```sh
kubectl get jobs --namespace your-namespace -o name | grep 'foo-bar' | xargs kubectl delete --namespace your-namespace
```

将 `your-namespace` 替换为实际的命名空间名称。


# k8s 如何设置默认 storageClass

在 Kubernetes 中，设置默认的 StorageClass 可以通过以下步骤实现：

1. **查看现有的 StorageClass：**

   首先，查看集群中已经存在的 StorageClass：

   ```sh
   kubectl get storageclass
   ```

2. **设置默认 StorageClass：**

   假设你想将某个名为 `my-storage-class` 的 StorageClass 设置为默认的，你需要编辑该 StorageClass 的配置，并添加 `storageclass.kubernetes.io/is-default-class` 注解。

   ```sh
   kubectl patch storageclass my-storage-class -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
   ```

3. **移除其他 StorageClass 的默认注解：**

   如果之前已经有其他 StorageClass 被设置为默认，你需要移除它们的默认注解。假设另一个名为 `old-default-storage-class` 的 StorageClass 之前被设置为默认：

   ```sh
   kubectl patch storageclass old-default-storage-class -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
   ```

4. **验证设置：**

   再次查看 StorageClass 列表，确认新的默认 StorageClass 设置成功：

   ```sh
   kubectl get storageclass
   ```

   你应该会看到 `my-storage-class` 的 `storageclass.kubernetes.io/is-default-class` 注解被设置为 `true`。

通过这些步骤，你可以成功地设置或更改 Kubernetes 集群中的默认 StorageClass。